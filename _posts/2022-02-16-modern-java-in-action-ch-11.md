---
title: "Modern Java in Action 11장; null 대신 Optional 클래스"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 11. null 대신 Optional 클래스

#### 11.1 값이 없는 상황을 어떻게 처리할까?
##### 11.1.1 보수적인 자세로 NullPointerException 줄이기
* 변수가 `null` 인지 확인하기 위해 중첩 `if` 블록을 사용하면 코드 들여쓰기 수준이 증가하고 구조와 가독성에 나쁜 영향을 미친다.
* `null`마다 실패를 반환하게 하면 함수에 여러 출구가 생겨 유지 보수가 어려워진다.

##### 11.1.2 null 때문에 발생하는 문제
* 에러의 근원이다.
* 코드를 어지럽힌다.
* 아무 의미가 없다.
* 자바 철학에 위배된다: 자바는 개발자로부터 모든 포인터를 숨겼지만 `null`은 예외이다.
* 형식 시스템에 구멍을 만든다: `null`은 무형식이며 정보를 포함하고 있지 않아 모든 참조 형식에 할당할 수 있다. 다른 부분으로 퍼지면 어떤 의미로 사용되었는지 알 수 없다.

##### 11.1.3 다른 언어는 null 대신 무얼 사용하나?
* 안전 네비게이션 연산자(`?.`)를 도입해서 호출 체인에 `null`이 있으면 결과로 `null`이 반환된다.
* 선택형 값을 저장할 수 있는 형식을 도입해서 아무 값도 갖지 않을 수 있도록 한다. 이 형식에서 제공하는 연산을 이용해 값이 있는지 여부를 명시적으로 확인해야 한다.
  * 자바는 이에 영향을 받아 `Optional`을 제공한다.

#### 11.2 Optional 클래스 소개
* `Optional`은 선택형값을 캡슐화하는 클래스다.
  * 값이 있으면 `Optional` 클래스는 값을 감싼다.
  * 값이 없으면 `Optional.empty` 메서드로 `Optional`을 반환한다. 이 메서드는 `Optional`의 특별한 싱글턴 인스턴스를 반환하는 정적 팩토리 메서드다.
* `Optional`을 이용하면 값이 엇는 상황이 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다.
  * 모든 `null` 참조를 `Optional`로 대체하는 것이 아니라 더 이해하기 쉬운 API를 설계하는 데에 사용할 수 있다. 즉, 메서드의 시그니처만 보고도 선택형값인지 구분할 수 있다.

#### 11.3 Optional 적용 패턴
##### 11.3.1 Optional 객체 만들기
* `Optional.empty`로 빈 `Optional` 객체를 얻을 수 있다.
* `Optional.of`로 `null`이 아닌 값을 포함하는 객체를 얻을 수 있다.
* `Optional.ofNullable`로 `null` 값을 저장하는 객체를 얻을 수 있다.

###### 11.3.2 맵으로 Optional의 값을 추출하고 변환하기
* 스트림의 `map` 메서드와 유사한 `map` 메서드를 지원한다.
```java
Optional<String> name = optInsurance.map(Insurance::getName);
```

##### 11.3.3 flatMap으로 Optional 객체 연결
* 한 `Optional` 객체에 대해 `map` 메서드를 중첩해서 위처럼 사용할 수 없다. `Optional` 객체가 중첩되기 때문이다.
* 스트림에서 `flatMap`은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다. 즉, 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화된다. 
  * `Optional`도 이와 유사하게 이차원 `Optional`을 `flatMap`으로 평준화할 수 있다.  
![](https://user-images.githubusercontent.com/55083845/154683570-b81f069b-0e30-413c-8810-0398152a9774.jpeg)

```java
String carInsuranceName = person.flatMap(Person::getCar)
                                .flatMap(Car::getInsurance)
                                .map(Insurance::getName)
                                .orElse("unknown");
```
![](https://user-images.githubusercontent.com/55083845/154683991-77df8128-444a-4f93-85e6-c12a04c2a7d3.jpeg)
* `flatMap`의 첫 번째 단계에서는 `Optional` 내부의 객체에 함수를 ㅈ거용한다. 위의 예에서 `flatMap(Person::getCar)`의 결과는 `Optional<Car>`를 반환하므로 중첩된 `Optional`이 생긴다.
  * 이를 `flatMap`으로 평준화한다. 두 `Optional`을 합치면서 둘 중 하나라도 `null`이면 빈 `Optional` 객체를 생성한다.
* `Optional`이 빈 경우 기본값을 제공하는 `orElse` 메서드를 사요할 수 있다.

##### 11.3.4 Optional 스트림 조작
```java
Set<String> carInsuranceNames = persons.stream()
                                       .map(Person::getCar)
                                       .map(optCar -> optCar.flatMap(Car::getInsurance))
                                       .map(optIns -> optIns.map(Insurance::getName))
                                       .flatMap(Optional::stream) 
                                       // Stream<Optional<String>>을 Stream<String>으로
                                       .collect(toSet());
```
* `Optional.stream()`은 각 `Optional`이 비어 있는지 아닌지에 따라 `Optional`을 0개 이상의 항목을 포함하는 스트림으로 변환한다. 따라서 이 메서드의 참조를 스트림의 한 요소에서 다른 스트림으로 적용하는 함수로 볼 수 있으며 이를 원래 스트림에 호출하는 `flatMap` 메서드로 전달할 수 있다.

##### 11.3.5 디폴트 액션과 Optional 언랩
* `get`: 래핑된 값이 있으면 해당 값을 반환하고 없으면 예외를 발생시킨다.
* `orElse`: 값을 포함하지 않을 때 기본값을 제공한다.
* `orElseGet(Supplier<? extends T> other)`: `Optional`에 값이 없을 때만 `Supplier`가 실행된다.
* `orElseThrow(Supplier<? extends X> exceptionSupplier)`: `Optional`이 비어 있을 때 예외를 발생시킨다.
* `ifPresent(Consumer<? super T> consumer)`: 값이 존재할 때 인수로 넘겨 준 동작을 실행할 수 있다.
* `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`: `Optional`이 비었을 때 실행할 수 있는 `Runnable`을 받는다.

##### 11.3.7 필터로 특정값 거르기
* `Optional` 객체에 `filter` 메서드를 사용하면 해당 객체가 값을 가지고 프레디케이트와 일치하는 경우에 `filter` 메서드는 그 값을 반환하고 그렇지 않으면 빈 `Optional` 객체를 반환한다.
  * `Optional`은 최대 한 개의 요소를 포함할 수 있는 스트림과 같으므로 비어 있다면 `filter` 연산은 아무 동작도 하지 않고 값이 있다면 그 값에 프레디케이트를 적용한다. 
  * 프레디케이트 적용 결과가 true면 `Optional`에는 아무 변화도 일어나지 않는다. 하지만 false면 값은 사라져 버리고 `Optional`은 빈 상태가 된다.

#### 11.4 Optional을 사용한 실용 예제
##### 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기
* `Optional<Object> value = Optional.ofNullable(map.get("key"));`

##### 11.4.2 예외와 Optional 클래스
* 값을 제공할 수 없을 때 예외를 발생시키는 경우 `Optional`을 반환하도록 유틸리티 메서드를 구현할 수 있다.
```java
public static Optional<Integer> stringToInt(String s) {
  try {
    return Optional.of(Integer.parseInt(s));
  } catch (NumberFormatException e) {
    return Optional.empty();
  }
}
```

##### 11.4.3 기본형 Optional을 사용하지 말아야 하는 이유
* `Optional`의 최대 요소 수는 한 개이므로 기본형 스트림처럼 기본형 특화 클래스를 이용한다고 해서 성능을 개선할 수 없다.
* 또한 기본형 특화 클래스의 경우 `Optional` 클래스의 유용한 메서드들을 지원하지 않는다.
* 기본형 특화 클래스로 생성한 결과는 다른 일반 `Optional`과 혼용할 수 없다.
