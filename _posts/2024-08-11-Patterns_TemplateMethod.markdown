---
layout: post
title:  "템플릿 메소드 패턴(Template Method Pattern)"
date:   2024-08-11 16:37:00 +0900
categories: patterns
---

템플릿 메소드 패턴(Template Method Pattern)은 객체지향 설계에서 사용하는 행동 패턴 중 하나로, 알고리즘의 구조를 정의하고, 알고리즘의 일부 단계를 하위 클래스에서 재정의할 수 있도록 허용하는 패턴입니다. 이 패턴을 사용하면 알고리즘의 공통된 부분은 상위 클래스에서 관리하고, 변화하는 부분은 하위 클래스에서 구현할 수 있습니다.

## 주요 개념:
추상 클래스(Abstract Class): 알고리즘의 뼈대를 정의하는 역할을 합니다. 이 클래스에는 템플릿 메소드가 있으며, 이 메소드는 알고리즘의 단계들을 호출합니다. 일부 단계는 하위 클래스에서 구현되어야 하므로 추상 메소드로 정의됩니다.

템플릿 메소드(Template Method): 알고리즘의 구조를 정의하는 메소드입니다. 이 메소드 안에서 알고리즘의 각 단계를 호출하며, 일부 단계는 하위 클래스에서 구체적으로 구현됩니다.

구체 클래스(Concrete Class): 추상 클래스의 하위 클래스로, 추상 메소드를 구현하여 알고리즘의 특정 단계를 정의합니다.

## 장점:
코드 재사용성: 알고리즘의 공통된 부분을 상위 클래스에 두고, 중복을 최소화할 수 있습니다.
유지보수성: 알고리즘의 구조가 변경될 때, 상위 클래스에서 한 번만 수정하면 되므로 유지보수가 용이합니다.
확장성: 새로운 하위 클래스를 만들어 추가적인 기능을 쉽게 구현할 수 있습니다.

## 단점:
상속의 한계: 템플릿 메소드 패턴은 상속을 기반으로 하기 때문에, 상속의 한계나 다중 상속의 문제점이 발생할 수 있습니다.
구조의 복잡성: 알고리즘이 복잡해질수록 상위 클래스와 하위 클래스 간의 관계가 복잡해질 수 있습니다.

## 예시:
커피와 차를 만드는 과정에서 템플릿 메소드 패턴을 적용한 예시를 보겠습니다.

### 1. 추상 클래스
커피와 차를 만드는 공통된 절차를 정의한 추상 클래스입니다.

```java
abstract class Beverage {
    // 템플릿 메소드
    public final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    // 공통 단계
    private void boilWater() {
        System.out.println("Boiling water");
    }

    private void pourInCup() {
        System.out.println("Pouring into cup");
    }

    // 변하는 단계는 하위 클래스에서 구현
    protected abstract void brew();

    protected abstract void addCondiments();
}
```

### 2. 구체 클래스들
커피와 차를 만드는 구체적인 과정을 정의한 클래스들입니다.

```java
class Tea extends Beverage {
    @Override
    protected void brew() {
        System.out.println("Steeping the tea");
    }

    @Override
    protected void addCondiments() {
        System.out.println("Adding lemon");
    }
}

class Coffee extends Beverage {
    @Override
    protected void brew() {
        System.out.println("Dripping coffee through filter");
    }

    @Override
    protected void addCondiments() {
        System.out.println("Adding sugar and milk");
    }
}
```

### 3. 사용 예시
이제 커피와 차를 준비하는 코드를 작성해보겠습니다.

```java
public class Main {
    public static void main(String[] args) {
        Beverage tea = new Tea();
        tea.prepareRecipe();
        System.out.println();

        Beverage coffee = new Coffee();
        coffee.prepareRecipe();
    }
}
```

실행 결과:

```sql
Boiling water
Steeping the tea
Pouring into cup
Adding lemon

Boiling water
Dripping coffee through filter
Pouring into cup
Adding sugar and milk
```

## 요약
템플릿 메소드 패턴은 알고리즘의 구조를 상위 클래스에 정의하고, 구체적인 구현은 하위 클래스에서 제공하도록 하는 패턴입니다. 이를 통해 코드의 재사용성을 높이고 유지보수성을 개선할 수 있으며, 다양한 상황에서 유연하게 알고리즘을 확장할 수 있습니다.
