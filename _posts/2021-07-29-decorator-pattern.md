---
title: "[Design Pattern] Decorator Pattern 알아보기"
categories:
  - Develop
tags:
  - "design-pattern"
  - decorator
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 데코레이터 패턴에 대해 알아보기

#### 데코레이터 패턴?

* 기본 기능에 여러 가지 기능을 추가할 수 있는 경우에, 각각의 추가 기능을 Decorator 클래스로 정의한 후 필요한 Decorator 객체를 조합함으로써 추가 기능의 조합을 설계하는 패턴이다.

![](https://www.programmergirl.com/wp-content/uploads/2019/08/DecoratorPattern.png)

* 기본 기능 + 여러 가지의 추가 기능에서 파생되는 다양한 조합을 동적으로 구현할 수 있다.
* 위의 그림에서 각각의 요소들이 수행하는 작업은 아래와 같다.
  * `Component`: 기본 기능인 `ConcreteComponent` 와 추가 기능인 `Decorator` 의 공통 기능을 정의한다. 
  * `ConcreteComponent`: 기본 기능을 구현하는 클래스이다.
  * `Decorator`: 구체적인 `Decorator` (부가 기능)들을 추상화한, 즉 구체적인 `Decorator`의 공통 기능을 제공한다.
  * `ConcreteDecorator1`, `ConcreteDecorator2`: `Decorator`의 하위 클래스로 기본 기능에 추가되는 부가적인 개별 기능들을 뜻한다. 이들은 `ConcreteComponent` 객체에 대한 참조가 필요한데, 이는 `Decorator` 클래스에서 `Component` 클래스로의 합성 관계를 통해 표현된다. 

#### 데코레이터 패턴의 예시 - 카페 음료

```java
public abstract class Beverage {
  String description = "beverage";
  
  public String getDescription() {
    return description;
  }
  
  public abstract double cost();
}
```

* 위와 같은 예시를 생각해 보자. 기본 클래스는 `Beverage` 이고, 서브 클래스에서 구체적인 음료와 가격을 정의하는 상황에서 단순히 상속으로 모든 것을 표현하는 경우 조합에 따라 수만가지의 클래스가 탄생할 것이다.

```java
public class Espresso extends Beverage {
  
  public Espresso() {
    description = "Espresso";
  }
  
  public double cost() {
    return 1.89;
  }
}
```

```java
public class EspressoWithMilk extends Beverage {
  
  public Espresso() {
    description = "Espresso with milk";
  }
  
  public double cost() {
    return 2.09;
  }
}
...
```

* 이를 피하기 위해 인스턴스 변수를 추가하고 상속을 사용해 추가 사항을 관리한다고 가정해 보자.

```java
public class Beverage {
  String description = "beverage";
  boolean milk = false;
  boolean soy = false;
  boolean whip = false;
  
  ...
  
  public String getDescription() {
    return description;
  }
  
  public boolean getSoy() {
    return soy;
  }
  
  // getter, setter...
  
  public double cost() {
    double cost = 0; 
    if(getSoy()) {
      cost += 0.4;
    }
    ...
    return cost;
  }
}
```

* 기본 클래스에서는 추가된 항목의 가격을 계산하고, 서브클래스에서는 `Beverage` 의 `cost()` 와 자신의 음료 값을 더한다.
* 전보다는 단순해졌지만 첨가물의 가격이나 종류 등이 바뀌면 기존 코드를 수정해야 한다. 또 휘핑을 두 번 추가하는 경우 등 다양한 조합에 대해서 기존 코드를 수정하거나 새로운 클래스를 생성하게 된다

> 데코레이터 패턴을 적용하면 어떻게 될까?

* 특정 음료에서 시작해서 첨가물로 음료를 장식한다고 생각해 보자. 음료 객체를 가져오고, 첨가물 객체로 그 객체를 장식하는 행위를 계속한다.  `cost()` 를 호출하면 첨가물의 가격을 계산하는 일은 해당 객체들에게 위임된다.
  * 데코레이터 객체를 래퍼 객체로 생각해 보자. 
  * 마지막으로 감싼 객체의 `cost()`는 그 전 객체의 `cost()` 를 호출하고 그렇게 호출을 반복하면 특정 음료로 돌아온다. 특정 음료에서 가격을 반환하면 다시 반대 순서로 반환되면서 가격을 더해 준다.

* 위의 데코레이터 패턴의 구성 요소들을 카페 음료 예시에 대입하면 다음과 같다.
  * `Component`: `Beverage`
  * `ConcreteComponent`: `Espresso`, ...
  * `Decorator`: `CondimentDecorator` - 첨가물의 공통 기능
  * `ConcreteDecorator1`,`ConcreteDecorator2`: `Milk`, `Soy`, ...

```java
public abstract class Beverage {
  String description = "beverage";
  
  public String getDescription() {
    return description;
  }
  
  public abstract double cost();
}
```

```java
public abstract class CondimentDecorator extends Beverage {
  public abstract String getDescription();
}
```

```java
public class Espresso extends Beverage {
  
  public Espresso() {
    description = "Espresso";
  }
  
  public double cost() {
    return 1.89;
  }
}
```

```java
public class Mocha extends CondimentDecorator {
  Beverage baverage;
  
  public Mocha(Beverage beverage) {
    this.baverage = baverage;
  }
  
  public String getDescription() {
    return baverage.getDescription() + "mocha";
  }
  
  public double cost() {
    return 0.20 + beverage.cost();
  }
}
```

* `ConcreteDecorator` 는 `ConcreteComponent` 객체에 대한 참조가 필요한데, 이는 `Decorator` 클래스에서 `Component` 클래스로의 합성 관계를 통해 표현된다.
  * 위의 예시에서는 `ConcreteDecorator` 인 `Mocha` 가 생성자에서 `ConcreteComponent` 인 `Beverage` 필드에 대한 객체를 생성하고 있다. 
  * 좀 더 정확히 말하면 `Beverage` 는 `Component` 지만, 모든 `ConcreteComponent` 의 슈퍼 클래스이므로 모든 `ConcreteComponent` 를 객체로 가질 수 있도록 위와 같이 작성된다.
  * 추가되는 가격만 알고 싶다면 추가되는 가격만 반환하는 메소드를 새로 작성할 수 있다.
* 처음 작성한 코드보다 훨씬 변화에 유연하고 일관적인 코드를 작성할 수 있다.

* 이렇게 Decorator 패턴을 사용하면 기본 기능에 더해 추가 기능을 조합해서 일관적이고 간편하게 기본 기능과 추가 기능의 조합을 구현할 수 있다.

* 사용 예시는 아래와 같다.

```java
public class Customer {
  public static void main(String[] args) {
    Beverage myBeverage = 
      new Mocha(
    	new Whip(
      new DarkRoast()));
    myBeverage.cost();
  }
}
```



#### 참고

* [Head First Design Patterns](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=582754)
* https://gmlwjd9405.github.io/2018/07/09/decorator-pattern.html
