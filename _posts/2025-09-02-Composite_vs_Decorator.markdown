---
layout: post
title:  "Composite vs Decorator"
date:   2025-09-02 13:04:00 +0900
categories: patterns
---

두 패턴은 같은 인터페이스를 공유한다는 공통점이 있지만 목적이 완전히 다릅니다.

## 핵심 차이 (한눈에)

|	구분	|	Composite	|	Decorator	|
|	---	|	---	|	---	|
|	목적	|	부분-전체(트리) 구조를 하나의 객체처럼 다루기	|	객체에 기능을 동적으로 덧입히기	|
|	구성	|	여러 자식을 보유(Composite 노드) 또는 자식 없음(Leaf)	|	단일 래핑 대상을 보유(보통 1개)	|
|	전형적 연산	|	합/평균/일괄처리처럼 재귀적 집계	|	호출 전/후 부가 로직(로깅, 캐시, 권한, 수수료 등)	|
|	사용 시점	|	도메인 모델링(구조)	|	런타임 행동 확장(기능)	|
|	예	|	카테고리–상품 트리	|	가격에 쿠폰/세금/로그를 중첩 적용	|

## Java 예시 1) Composite — 카테고리/상품 트리

```java
// Composite 공통 인터페이스
import java.math.BigDecimal;
import java.util.*;

interface CatalogComponent {
    BigDecimal price();         // 이 노드(하위 포함)의 총합
    void print(String indent);  // 구조 출력

    // 기본적으로 Leaf는 add/remove 미지원
    default void add(CatalogComponent c) { throw new UnsupportedOperationException(); }
    default void remove(CatalogComponent c) { throw new UnsupportedOperationException(); }
}

// Leaf: 실제 상품
class Product implements CatalogComponent {
    private final String name;
    private final BigDecimal unitPrice;

    public Product(String name, BigDecimal unitPrice) {
        this.name = name;
        this.unitPrice = unitPrice;
    }
    public BigDecimal price() { return unitPrice; }
    public void print(String indent) {
        System.out.println(indent + "- " + name + " : " + unitPrice);
    }
}

// Composite: 카테고리(여러 자식 보유)
class Category implements CatalogComponent {
    private final String name;
    private final List<CatalogComponent> children = new ArrayList<>();

    public Category(String name) { this.name = name; }

    public void add(CatalogComponent c) { children.add(c); }
    public void remove(CatalogComponent c) { children.remove(c); }

    public BigDecimal price() {
        return children.stream()
                .map(CatalogComponent::price)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public void print(String indent) {
        System.out.println(indent + "[" + name + "]");
        for (CatalogComponent c : children) c.print(indent + "  ");
    }
}

// Demo
class CompositeDemo {
    public static void main(String[] args) {
        Category root = new Category("Store");
        Category books = new Category("Books");
        books.add(new Product("Clean Code", new BigDecimal("25.00")));
        books.add(new Product("DDD", new BigDecimal("30.00")));
        Category electronics = new Category("Electronics");
        electronics.add(new Product("Mouse", new BigDecimal("15.00")));
        electronics.add(new Product("Keyboard", new BigDecimal("40.00")));

        root.add(books);
        root.add(electronics);

        root.print("");
        System.out.println("TOTAL: " + root.price()); // 트리 전체 합계
    }
}
```

* 포인트: 클라이언트는 Product(Leaf)와 Category(Composite)를 동일한 타입으로 취급하며, 합계 계산이 재귀적으로 이루어집니다.

## Java 예시 2) Decorator — 가격 계산에 쿠폰/세금/로깅 덧입히기

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.*;

// 공통 인터페이스
interface Pricing {
    BigDecimal total(List<Item> items);
}

class Item {
    final String sku;
    final BigDecimal unitPrice;
    final int qty;
    Item(String sku, BigDecimal unitPrice, int qty) {
        this.sku = sku; this.unitPrice = unitPrice; this.qty = qty;
    }
}

// 기본 구현
class BasicPricing implements Pricing {
    public BigDecimal total(List<Item> items) {
        return items.stream()
                .map(i -> i.unitPrice.multiply(BigDecimal.valueOf(i.qty)))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// 데코레이터 베이스
abstract class PricingDecorator implements Pricing {
    protected final Pricing inner;
    protected PricingDecorator(Pricing inner) { this.inner = inner; }
}

// 쿠폰(퍼센트 할인)
class CouponDecorator extends PricingDecorator {
    private final BigDecimal off; // 0.10 = 10% off
    public CouponDecorator(Pricing inner, BigDecimal off) { super(inner); this.off = off; }
    public BigDecimal total(List<Item> items) {
        BigDecimal base = inner.total(items);
        return base.multiply(BigDecimal.ONE.subtract(off));
    }
}

// 세금(부가세 추가)
class TaxDecorator extends PricingDecorator {
    private final BigDecimal rate; // 0.10 = 10% VAT
    public TaxDecorator(Pricing inner, BigDecimal rate) { super(inner); this.rate = rate; }
    public BigDecimal total(List<Item> items) {
        BigDecimal base = inner.total(items);
        return base.multiply(BigDecimal.ONE.add(rate));
    }
}

// 로깅
class LoggingDecorator extends PricingDecorator {
    public LoggingDecorator(Pricing inner) { super(inner); }
    public BigDecimal total(List<Item> items) {
        long t0 = System.currentTimeMillis();
        BigDecimal res = inner.total(items);
        System.out.println("[pricing] " + (System.currentTimeMillis()-t0) + "ms, total=" + res);
        return res;
    }
}

// Demo
class DecoratorDemo {
    public static void main(String[] args) {
        List<Item> cart = List.of(
            new Item("A1", new BigDecimal("100.00"), 2),
            new Item("B2", new BigDecimal("50.00"), 1)
        );

        Pricing pricing =
            new LoggingDecorator(
                new TaxDecorator(
                    new CouponDecorator(
                        new BasicPricing(),
                        new BigDecimal("0.10") // 10% 할인
                    ),
                    new BigDecimal("0.10")     // 10% 세금
                )
            );

        BigDecimal amount = pricing.total(cart).setScale(2, RoundingMode.HALF_UP);
        System.out.println("PAYABLE: " + amount);
    }
}
```

* 포인트: Pricing 인터페이스는 그대로이고, 래핑 순서에 따라 기능이 조합됩니다. 필요할 때만 데코레이터를 동적으로 중첩하면 돼요.

### 언제 무엇을 쓰나

* 트리/부분-전체 모델링이 필요하면 Composite
  (카테고리–상품, 조직도, 장바구니 묶음상품 등)

* 기능을 선택적으로 추가/제거하고 싶으면 Decorator
  (로깅·캐시·권한·세금/쿠폰/배송비 등)


---


2-1. Composite 패턴 예시(도형 객체를 그룹화하여 동시에 그리기)java // Componentinterface Graphic {    void draw();}// Leaf - 원class Circle implements Graphic {    @Override    public void draw() {        System.out.println("원을 그립니다.");    }}// Leaf - 사각형class Rectangle implements Graphic {    @Override    public void draw() {        System.out.println("사각형을 그립니다.");    }}// Composite — 여러 Graphic을 모아놓은 그룹class CompositeGraphic implements Graphic {    private java.util.List<Graphic> children = new java.util.ArrayList<>();    public void add(Graphic graphic) {        children.add(graphic);    }    public void remove(Graphic graphic) {        children.remove(graphic);    }    @Override    public void draw() {        for (Graphic g : children) {            g.draw();        }    }}// 메인public class CompositePatternExample {    public static void main(String[] args) {        Circle circle = new Circle();        Rectangle rectangle = new Rectangle();        CompositeGraphic group = new CompositeGraphic();        group.add(circle);        group.add(rectangle);        group.draw(); // -> 원, 사각형 둘 다 출력    }}🔹 특징: 단일 객체(Leaf)와 그룹(Composite)을 동일하게 draw()로 다룰 수 있음.2-2. Decorator 패턴 예시(출력 메시지에 동적으로 기능 추가)java // Componentinterface Printer {    void print();}// Concrete Componentclass SimplePrinter implements Printer {    @Override    public void print() {        System.out.print("Hello");    }}// Decoratorabstract class PrinterDecorator implements Printer {    protected Printer printer;    public PrinterDecorator(Printer printer) {        this.printer = printer;    }    @Override    public void print() {        printer.print(); // 기본 기능 위임    }}// Concrete Decorator — 감싸서 기능 확장class ExclamationDecorator extends PrinterDecorator {    public ExclamationDecorator(Printer printer) {        super(printer);    }    @Override    public void print() {        super.print();        System.out.print("!");    }}class BracketDecorator extends PrinterDecorator {    public BracketDecorator(Printer printer) {        super(printer);    }    @Override    public void print() {        System.out.print("[");        super.print();        System.out.print("]");    }}// 메인public class DecoratorPatternExample {    public static void main(String[] args) {        Printer printer = new SimplePrinter();        Printer decorated =            new BracketDecorator(                new ExclamationDecorator(printer)            );        decorated.print(); // 출력: [Hello!]    }}


2-1. Composite 패턴 예시(도형 객체를 그룹화하여 동시에 그리기)java // Componentinterface Graphic {    void draw();}// Leaf - 원class Circle implements Graphic {    @Override    public void draw() {        System.out.println("원을 그립니다.")
   }}// Leaf - 사각형class Rectangle implements Graphic {    @Override    public void draw() {        System.out.println("사각형을 그립니다.")
   }}// Composite — 여러 Graphic을 모아놓은 그룹class CompositeGraphic implements Graphic {    private java.util.List<Graphic> children = new java.util.ArrayList<>()
   public void add(Graphic graphic) {        children.add(graphic)
   }    public void remove(Graphic graphic) {        children.remove(graphic)
   }    @Override    public void draw() {        for (Graphic g : children) {            g.draw()
       }    }}// 메인public class CompositePatternExample {    public static void main(String[] args) {        Circle circle = new Circle()
       Rectangle rectangle = new Rectangle()
       CompositeGraphic group = new CompositeGraphic()
       group.add(circle)
       group.add(rectangle)
       group.draw()
// -> 원, 사각형 둘 다 출력    }}🔹 특징: 단일 객체(Leaf)와 그룹(Composite)을 동일하게 draw()로 다룰 수 있음.2-2. Decorator 패턴 예시(출력 메시지에 동적으로 기능 추가)java // Componentinterface Printer {    void print();}// Concrete Componentclass SimplePrinter implements Printer {    @Override    public void print() {        System.out.print("Hello")
   }}// Decoratorabstract class PrinterDecorator implements Printer {    protected Printer printer
   public PrinterDecorator(Printer printer) {        this.printer = printer
   }    @Override    public void print() {        printer.print()
// 기본 기능 위임    }}// Concrete Decorator — 감싸서 기능 확장class ExclamationDecorator extends PrinterDecorator {    public ExclamationDecorator(Printer printer) {        super(printer)
   }    @Override    public void print() {        super.print()
       System.out.print("!")
   }}class BracketDecorator extends PrinterDecorator {    public BracketDecorator(Printer printer) {        super(printer)
   }    @Override    public void print() {        System.out.print("[")
       super.print()
       System.out.print("]")
   }}// 메인public class DecoratorPatternExample {    public static void main(String[] args) {        Printer printer = new SimplePrinter()
       Printer decorated =            new BracketDecorator(                new ExclamationDecorator(printer)            )
       decorated.print()
// 출력: [Hello!]    }}