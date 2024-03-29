---
title: "Modern Java in Action 5장; 스트림 활용"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 5. 스트림 활용

#### 5.1 필터링

##### 5.1.1 프레디케이트로 필터링
* `filter` 메서드는 프레디케이트를 인수로 받아 일치하는 모든 요소를 포함하는 스트림을 반환한다.

##### 5.1.2 고유 요소 필터링
* `distinct` 메서드는 고유 요소로 이루어진 스트림을 반환한다. (고유 여부는 스트림에서 만든 객체의 `hashCode`, `equals`로 결정)

#### 5.2 스트림 슬라이싱

##### 5.2.1 프레디케이트를 이용한 슬라이싱
* `takeWhile`을 이용하면 스트림에 프레디케이트를 적용해 스트림을 특정 조건에 맞도록 슬라이스할 수 있다.
* 특정 조건의 나머지 요소를 선택하는, 즉 `takeWhile`과 반대되는 작업은 `dropWhile`을 사용한다.

##### 5.2.2 스트림 축소
* 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 `limit` 메서드를 사용할 수 있다.
* 정렬되지 않은 스트림에서도 사용 가능하다.

##### 5.2.3 요소 건너뛰기
* 처음 n개 요소를 제외한 스트림을 반환하는 `skip` 메서드가 있다.

#### 5.3 매핑

##### 5.3.1 스트림의 각 요소에 함수 적용하기
* 함수를 인수로 받는 `map` 메서드를 지원한다. 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.
* 여러 `map`을 연결할 수 있다.

##### 5.3.2 스트림 평면화
* `flatMap`은 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

#### 5.4 검색과 매칭

##### 5.4.1 프레디케이트가 적어도 한 요소와 일치하는지 확인
* `anyMatch` 메서드를 이용한다. 불리언을 반환하므로 최종 연산이다.

##### 5.4.2 프레디케이트가 모든 요소와 일치하는지 검사
* `allMatch`를 사용한다. 불리언을 반환하므로 최종 연산이다.
* `noneMatch`는 이와 반대인 연산이다.

> `anyMatch`, `allMatch`, `noneMatch` 메서드는 스트림 쇼트서킷 기법을 활용한다. 표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 된다.

##### 5.4.3 요소 검색
* `findAny`는 현재 스트림에서 임의의 요소를 반환한다.

###### Optinal이란?
* `findAny`는 아무 요소도 반환하지 않을 수 있으므로 `Optional<T>`을 결과로 반환한다.

##### 5.4.4 첫 번째 요소 찾기
* `findFirst`는 첫 번째 요소를 반환한다. 조건에 맞는 요소가 없다면 `null`일 수 있으므로 `Optional<T>`를 반환한다.

#### 5.5 리듀싱
* 스트림의 모든 요소를 반복적으로 처리하는 질의를 **리듀싱 연산**이라고 한다.
* 함수형 프로그래밍 언어 용어로는 이 과정이 마치 종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미로 폴드라고 한다.

##### 5.5.1 요소의 합
* `int sum = numbers.stream().reduce(0, (a, b) -> a + b);`
* `reduce`는 두 개의 인수를 가진다.
  * 초깃값
  * 두 요소를 조합해서 새로운 값을 만드는 `BinaryOperator<T>`
  * 이전의 결과값이 람다의 첫 번째 파라미터, 다음 요소가 람다의 두 번째 파라미터가 된다.
* 초깃값을 받지 않도록 오버로드된 `reduce`는 `Optional` 객체를 반환한다.

##### 5.5.2 최댓값과 최솟값
* `reduce` 연산은 새로운 값을 이용해서 스트림의 모든 요소를 소비할 때까지 람다를 반복 수행하며 최댓값을 생산한다.
* `map`과 `reduce`를 연결하는 기법을 맵 리듀스 패턴이라 하며, 쉽게 병렬화하는 특징이 있다.
```java
int count = 
    menu.stream().
        .map(d -> 1)
        .reduce(0, (a, b) -> a + b);
```
> `reduce`를 이용하면 내부 반복이 추상화되면서 내부에서 병렬로 `reduce`를 실행할 수 있게 된다. 반복적인 합계에서는 `sum` 변수를 공유해야 하므로 쉽게 병렬화하기 어렵다.

###### 스트림 연산: 상태 없음과 상태 있음
* `map`, `filter` 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다. 따라서 사용자가 제공한 람다나 메서드 참조가 내부적인 가변 상태를 가지지 않으면 보통 상태가 없는, 내부 상태를 갖지 않는 연산이 된다.
* 하지만 `reduce`, `sum`, `max` 같은 연산은 결과를 누적할 내부 상태가 필요하다. 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정되어 있다.
* `sorted`나 `distinct`는 스트림을 입력받아 다른 스트림을 출력하는 것처럼 보일 수 있지만 스트림의 요소를 정렬하거나 중복을 제거하려면 과거의 이력을 알고 있어야 한다. 어떤 요소를 출력 스트림으로 추가하려면 모든 요소가 버퍼에 추가되어 있어야 한다. 이러한 연산을 **내부 상태를 갖는 연산**이라 한다.


#### 5.7 숫자형 스트림

##### 5.7.1 기본형 특화 스트림
* 스트림 API는 박싱 비용을 피할 수 있도록 `int` - `IntStream`, `double` - `DoubleStream`, `long` - `LongStream` 을 제공한다.
* 스트림을 특화 스트림으로 변환할 때는 `mapToInt` (나머지도 마찬가지) 메서드를 가장 많이 사용한다.
*  `boxed` 메서드로 특화 스트림을 일반 스트림으로 변환할 수 있다. (`Stream<Integer> stream = intStream.boxed`)
* `OptionalInt`라는 기본형 특화 스트림 버전도 제공한다.

##### 5.7.2 숫자 범위
* `IntStream`과 `LongStream`은 `range`, `rangeClosed` 라는 두 가지 정적 메서드를 제공한다.


#### 5.8 스트림 만들기

##### 5.8.1 값으로 스트림 만들기
* 임의의 수를 인수로 받는 정적 메서드 `Stream.of` 를 이용해서 스트림을 만들 수 있다.

##### 5.8.2 null이 될 수 있는 객체로 스트림 만들기
* `Stream.ofNullable`을 이용해 만들 수 있다.

##### 5.8.3 배열로 스트림 만들기
* 배열을 인수로 받는 정적 메서드 `Arrays.stream`을 이용해서 스트림을 만들 수 있다.

##### 5.8.4 파일로 스트림 만들기
* `java.nio.files.Files`의 많은 정적 메서드가 스트림을 반환한다.
  * `Files.lines`는 주어진 파일의 행 스트림을 문자열로 반환한다.
* `Stream` 인터페이스는 `AutoCloseable` 인터페이스를 구현하므로 자원은 자동으로 관리된다.


##### 함수로 무한 스트림 만들기
* `Stream.iterate`와 `Stream.generate`는 함수에서 스트림을 만들 수 있다. 두 연산을 이용해서 무한 스트림을 만들 수 있다.
  * 이 두 메서드에서 만든 스트림은 요청할 때마다 주어진 함수를 이용해서 값을 만든다.
* `iterate` 메서드는 초깃값과 람다를 인수로 받아 새로운 값을 끊임없이 생산할 수 있는 반면 `generate`는 생산된 각 값을 연속적으로 계산하지 않는다.
* 무한 스트림의 요소는 무한적으로 계산이 반복되므로 정렬하거나 리듀스할 수 없다.