---
title: "Modern Java in Action 3장; 람다 표현식"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 3. 람다 표현식

#### 3.1 람다란 무엇인가?

* **람다 표현식**은 메서드로 전달할 수 있는 익명 함수를 단순화한 것
    * 익명: 보통의 메서드와 달리 이름이 없다.
    * 함수: 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
    * 전달: 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
    * 간결성: 익명 클래스처럼 많은 자질구레한 코드르 표현할 필요가 없다.
* 람다 표현식은 파라미터, 화살표, 바디로 이루어진다.
    * `(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());`
    * 람다 바디는 람다의 반환값에 해당하는 표현식이다. 파라미터 리스트에서 받은 파라미터를 사용할 수 있다.
    * `(parameters) -> expression` 혹은 `(parameters) -> { statemets; }`로 표현 가능하다.
    * 람다 표현식에는 `return`이 함축되어 있어 명시적으로 사용하지 않아도 된다.

#### 3.2 어디에, 어떻게 람다를 사용할까?

##### 함수형 인터페이스
* **함수형 인터페이스**는 **정확히 하나의 추상 메서드를 지정**하는 인터페이스이다.
* `Comparator`, `Runnable`, `Predicate<T>` 등이 있다.
* 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**할 수 있다.
    > 기술적으로로, 함수형 인터페이스를 구현한 클래스의 인스턴스

##### 함수 디스크립터
* 함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다. 람다 표현식의 시그니처를 서술하는 메서드를 **함수 디스크립터**라고 부른다.


#### 3.3 람다 활용: 실행 어라운드 패턴
* 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖춘 형식의 코드를 **실행 어라운드 패턴**이라고 부른다.
* 함수형 인터페이스를 이용해서 동작을 전달하도록 동작 파라미터화한다.
    * 함수형 인터페이스에 정의된 메서드(동작)의 시그니처화 일치하는 람다를 전달해서 동작을 실행한다.

#### 3.4 함수형 인터페이스 사용

##### 3.4.1 Predicate
* `test`라는 추상 메서드를 정의하며 제네릭 형식 `T`의 객체를 인수로 받아 불리언을 반환한다.

##### 3.4.2 Consumer
* `accept`라는 추상 메서드를 정의하며 제네릭 형식 `T`를 받아서 `void`를 반환한다.

##### 3.4.3 Function
* `apply`라는 추상 메서드를 정의하며 제네릭 형식 `T`를 인수로 받아 제네릭 형식 `R` 객체를 반환한다.

##### 기본형 특화
* 제네릭 파라미터에는 기본형이 아닌 참조형만 사용할 수 있다.
* 자바에서는 기본형을 참조형으로 반환하는 박싱 기능을 제공한다. (반대의 경우인 언박싱도 제공) 
* 박싱과 언박싱이 자동으로 이루어지는 오토박싱 기능도 제공한다.
* 하지만 변환 과정은 비용이 소모되는데, 자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.
    * `IntPredicate`, `DoublePredicate` 등
> 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않는다. 예외를 던지는 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 람다를 `try/catch` 블록으로 감싸야 한다.

#### 3.5 형식 검사, 형식 추론, 제약

##### 3.5.1 형식 검사
* 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등에서 기대되는 람다 표현식의 형식을 **대상 형식**이라고 부른다.
* 람다 표현식이 예외를 던질 수 있다면 추상 메서드도 같은 예외를 던질 수 있도록 `throws`로 선언해야 한다.

##### 3.5.2 같은 람다, 다른 함수형 인터페이스
* 대상 형식이라는 특징 때문에 같은 람다 표현식이더라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.
* 즉, 하나의 람다 표현식을 다양한 함수형 인터페이스에 사용할 수 있다.


##### 3.5.3 형식 추론
* 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다. 결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.


##### 3.5.4 지역 변수 사용
* 람다 표현식에서는 익명 함수가 하는 것처럼 **자유 변수**(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다.
* 이와 같은 동작을 **람다 캡처링**이라고 부른다.
* 자유 변수에도 약간의 제약이 있다.
    * 람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처할 수 있다.
    * 하지만 그러려면 지역 변수는 명시적으로 `final`로 선언되어 있어야 하거나 실질적으로 `final`로 선언된 변수와 똑같이 사용되어야 한다.
    * 즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.
> 인스턴스 변수는 힙에 저장되고 지역 변수는 스택에 위치한다. 람다에서 지역 변수로 바로 접근할 수 있다는 가정 하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근할 수 있다. 그러므로 자바 구현에서는 원래 변수에 접근을 허용하는 게 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생겼다.

#### 3.6 메서드 참조

##### 3.6.1 요약
* 메서드 참조를 이용하면 기존 메서드 구현으로 람다 표현식을 만들 수 있다. 이때 명시적으로 메서드 명을 참조함으로써 가독성을 높일 수 있다.
    * 메서드 명 앞에 구분자(`::`)를 붙이는 것으로 메서드 참조를 활용할 수 있다.
* 하나의 메서드를 참조하는 람다를 편리하게 표현할 수 있는 문법으로 간주할 수 있다.
* 메서드 참조는 세 가지 유형으로 구분할 수 있다.
    * 정적 메서드 참조: `Integer::parseInt`
    * 다양한 형식의 인스턴스 메서드 참조: `String::length`
    * 기존 객체의 인스턴스 메서드 참조: `expensiveTransaction`이라는 이름의 변수가 `Transaction` 객체를 할당받은 경우 `expensiveTransaction::getValue`, 비공개 헬퍼 메서드를 정의한 상황에서 그 메서드를 메서드 참조로 사용 가능
* 메서드 참조는 콘텍스트의 형식과 일치해야 한다.

##### 3.6.2 생성자 참조
* 클래스 명과 `new` 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다. 정적 메서드의 참조를 만드는 방법과 비슷하다.

#### 3.7 람다, 메서드 참조 활용하기
* 코드 전달
* 익명 클래스 사용
* 람다 표현식 사용
* 메서드 참조 사용

#### 3.8 람다 표현식을 조합할 수 있는 유용한 메서드
* 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있다. 이때 함수형 인터페이스가 제공하는 **디폴트 메서드**가 사용될 수 있다.
    * 추상 메서드가 아니므로 함수형 인터페이스의 정의를 벗어나지 않는다.

##### 3.8.1 Comparator 조합
```java
inventory.sort(comparing(Apple.getWeight()).reversed());
```

##### 3.8.2 Predicate 조합
* `Predicate` 인터페이스는 복잡한 형태를 만들 수 있도록 `negate`, `and` `or` 세 가지 메서드를 제공한다.
```java
Predicate<Apple> notRedApple = redApple.negate();
```

##### 3.8.3 Function 조합
* `Function` 인터페이스는 `Function` 인스턴스를 반환하는 `andThen`, `compose` 두 가지 디폴트 메서드를 제공한다.
```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 1;
Function<Integer, Integer> h = f.andThen(g);
int result = h.apply(1);
```