---
layout: post
title:  "전략 패턴(Strategy Pattern)"
date:   2024-08-11 11:32:00 +0900
categories: patterns
---

전략 패턴(Strategy Pattern)은 객체지향 설계에서 사용하는 행동 패턴 중 하나로, 알고리즘을 정의하고 각각의 알고리즘을 캡슐화하여 상호 교환이 가능하도록 만드는 패턴입니다. 이 패턴을 사용하면 클라이언트 코드와 알고리즘을 분리할 수 있어 코드의 유연성과 확장성을 높일 수 있습니다.

## 주요 개념:
Context(문맥): 클라이언트가 사용하는 인터페이스를 정의합니다. Context는 Strategy 객체를 포함하고, 이를 통해 특정 전략을 실행합니다.

Strategy(전략): 공통 인터페이스 또는 추상 클래스로, 알고리즘의 공통된 메서드를 정의합니다. 각기 다른 전략(알고리즘)은 이 인터페이스를 구현하거나 이 클래스를 상속받습니다.

Concrete Strategy(구체적인 전략): Strategy 인터페이스를 구현하여 실제 알고리즘을 정의하는 클래스들입니다. 여러 개의 Concrete Strategy가 존재할 수 있으며, 상황에 따라 이들을 선택하여 사용할 수 있습니다.

## 장점:
유연성: 알고리즘을 런타임에 쉽게 교체할 수 있습니다.
유지보수성: 알고리즘을 추가하거나 수정할 때 기존 코드를 변경할 필요가 없습니다. 새로운 알고리즘을 추가하면 됩니다.
단일 책임 원칙(SRP) 준수: 각각의 알고리즘은 독립된 클래스로 관리되기 때문에 코드가 더 깔끔해지고 관리하기 쉬워집니다.

## 단점:
복잡성 증가: 전략을 추가할 때마다 클래스 파일이 늘어날 수 있으며, 코드의 복잡성이 증가할 수 있습니다.
클라이언트의 책임 증가: 어떤 전략을 사용할지 클라이언트가 선택해야 하는 경우가 많아 클라이언트 코드가 복잡해질 수 있습니다.

## 예시:
온라인 상점에서 다양한 할인 전략을 사용할 수 있는 경우를 생각해 볼 수 있습니다. 할인 전략은 다양한 방식으로 구현될 수 있으며, 전략 패턴을 사용하면 이들 알고리즘을 쉽게 교체하거나 추가할 수 있습니다.

### 1. Strategy 인터페이스
먼저, 할인 전략을 나타내는 공통 인터페이스를 정의합니다.

```java
public interface DiscountStrategy {
    double applyDiscount(double amount);
}
```

### 2. Concrete Strategy들
구체적인 할인 전략을 구현한 클래스들을 정의합니다. 각각의 클래스는 DiscountStrategy 인터페이스를 구현합니다.

```java
// 할인이 없는 전략
public class NoDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(double amount) {
        return amount;
    }
}

// 퍼센트 할인을 적용하는 전략
public class PercentageDiscount implements DiscountStrategy {
    private double percentage;

    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }

    @Override
    public double applyDiscount(double amount) {
        return amount * (1 - percentage);
    }
}

// 고정된 금액을 할인하는 전략
public class FixedDiscount implements DiscountStrategy {
    private double discount;

    public FixedDiscount(double discount) {
        this.discount = discount;
    }

    @Override
    public double applyDiscount(double amount) {
        return Math.max(0, amount - discount);
    }
}
```

### 3. Context 클래스
ShoppingCart 클래스는 현재 사용 중인 할인 전략을 관리하며, 할인된 금액을 계산하는 역할을 합니다.

```java
public class ShoppingCart {
    private DiscountStrategy strategy;

    public ShoppingCart(DiscountStrategy strategy) {
        this.strategy = strategy;
    }

    public double calculateTotal(double amount) {
        return strategy.applyDiscount(amount);
    }

    // 필요에 따라 전략을 변경할 수 있는 메서드
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
}
```

### 4. 사용 예시
ShoppingCart 객체를 생성하고, 다양한 할인 전략을 적용하는 예시입니다.

```java
public class Main {
    public static void main(String[] args) {
        // 10% 할인 전략을 적용하는 장바구니
        ShoppingCart cart = new ShoppingCart(new PercentageDiscount(0.1));
        double total = cart.calculateTotal(100);
        System.out.println("Total with 10% discount: " + total); // 출력: 90.0

        // 고정 금액 20원 할인 전략으로 변경
        cart.setStrategy(new FixedDiscount(20));
        total = cart.calculateTotal(100);
        System.out.println("Total with 20 unit discount: " + total); // 출력: 80.0

        // 할인이 없는 전략으로 변경
        cart.setStrategy(new NoDiscount());
        total = cart.calculateTotal(100);
        System.out.println("Total with no discount: " + total); // 출력: 100.0
    }
}
```

## 요약
Strategy 인터페이스 (DiscountStrategy)는 공통된 메서드를 정의합니다.
Concrete Strategy (NoDiscount, PercentageDiscount, FixedDiscount)들은 이 인터페이스를 구현하여 각기 다른 할인 알고리즘을 제공합니다.
Context 클래스 (ShoppingCart)는 클라이언트가 사용하려는 전략을 관리하고, 이를 통해 금액 계산을 처리합니다.
이 구조를 통해 다양한 할인 전략을 유연하게 사용할 수 있으며, 새로운 할인 전략이 필요할 때도 기존 코드를 수정하지 않고 쉽게 추가할 수 있습니다.
