---
title: "Modern Java in Action 2장; 동작 파라미터화 코드 전달하기"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 2. 동작 파라미터화 코드 전달하기

#### 2.1 변화하는 요구사항에 대응하기
* 비슷한 코드가 반복 존재하면 그 코드를 추상화한다.

#### 2.2 동작 파라미터화
* 선택 조건을 결정하는 인터페이스(참, 거짓을 반환하는 함수 == 프레디케이트)를 생성해서 다양한 선택 조건을 대표하는 구현 클래스를 정의할 수 있다.
    * 이를 전략 디자인 패턴이라고 한다. 각 알고리즘(전략이라고 불리는)을 캡슐화하는 알고리즘 패밀리를 정의해 둔 다음 런타임에 알고리즘을 선택한다.
    * 메서드가 동작(또는 전략)을 받아서 내부적으로 다양한 동작을 수행할 수 있다.
* 동작을 파라미터화하면 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있다.

#### 2.3 복잡한 과정 간소화
* 클래스를 정의한 후에 인스턴스화하는 것은 번거롭다.
* 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스** 기법을 제공한다.

```java
List<Apple> = redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
})
```

* 람다를 사용하면 더욱 간단하게 구현할 수 있다.
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

#### 2.4 실전 예제
* `Comparator`로 정렬하기
* `Runnable`로 코드 블록 실행하기
* `Callable`을 결과로 반환하기
* GUI 이벤트 처리하기