---
title: "Modern Java in Action 20장; OOP와 FP의 조화: 자바와 스칼라 비교"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 20. OOP와 FP의 조화: 자바와 스칼라 비교
* 스칼라는 정적 형식의 프로그래밍 언어로 함수형의 기능을 수행하면서도 JVM에서 수행되는 언어이다.
* 복잡한 형식 시스템, 형식 추론, 패턴 매칭, 도메인 전용 언어를 단순하게 정의할 수 있는 구조 등을 제공한다. 모든 자바 라이브러리를 사용할 수 있다.

#### 20.1 스칼라 소개

##### 20.1.1 Hello beer

###### 명령형 스칼라

```scala
object Beer {
  def main(args: Array[String] {
    var n : int = 2
    while (n <= 6) {
      println(s"Hello ${n} bottles of beer")
      n += 1
    }
  })
}
```

* 자바에서는 클래스 내에 `main` 메서드를 선언했지만 스칼라에서는 `object`로 직접 싱글턴 객체를 만들 수 있다. 
  * `object`로 클래스를 정의하고 동시에 인스턴스화할 수 있다. 한 번에 단 하나의 인스턴스만 생성된다.
  * `object` 내부에 선언된 메서드는 정적 메서드로 간주할 수 있다.

###### 함수형 스칼라

```scala
obejct Beer {
  def main(args: Array[String]) {
    2 to 6 foreach { n => println(s"Hello ${n} bottles of beer") }
  }
}
```

* 스칼라에서는 모든 것이 객체이며 기본형이 없다. 2도 `Int` 형식의 객체이며 다른 `Int`를 인수로 받아 범위를 반환하는 `to`라는 메서드를 지원한다.
* 람다 표현식 문법은 자바와 비슷하다.

##### 20.1.2 기본 자료구조: 리스트, 집합, 맵, 튜플, 스트림, 옵션

###### 컬렉션 만들기

```scala
val authorsToAge = Map("R" -> 23, "M" -> 40, "A" -> 53)
```

* 스칼라는 자동으로 변수형을 추론하는 기능이 있다. 모든 변수의 형식은 컴파일 시 결정된다.
* `val`은 변수가 읽기 전용, 즉 변수에 값을 할당할 수 없음을 의미하며 `var`는 읽고 쓸 수 있는 변수를 가리킨다.

```java
val authors = List("R", "M", "A")
val numbers = Set(1, 1, 2, 3, 5, 7) // 다섯 개 요소 포함
```

###### 불변과 가변

* 일단 컬렉션을 만들고 나면 변경할 수 없다. 컬렉션이 불변이므로 언제 사용하든 항상 같은 요소를 갖게 되고 함수형 프로그래밍에서 유용하게 활용할 수 있다.
* 갱신해야 할 때는 기존 버전과 가능한 한 많은 자료를 공유하는 새로운 컬렉션을 만드는 방법으로 자료구조를 갱신한다.
  * 결과적으로 암묵적인 데이터 의존성을 줄일 수 있다.
* 패키지 `scala.collection.mutable`에서는 가변 버전의 컬렉션도 제공한다.

```java
Set<Integer> numbers = new HashSet<>();
Set<Integer> newNumbers = Collections.unmodifiableSet(numbers);
```

* 자바에서는 변경 불가 컬렉션을 만드는 다양한 방법을 제공한다. 위의 예에서 `newNumbers` 변수에는 새로운 요소를 추가할 수 없다.
  * 하지만 변경 불가 컬렉션은 값을 고칠 수 있는 컬렉션의 래퍼에 불과하므로 `numbers` 변수를 이용하면 새로운 요소를 추가할 수 있다.
* 반면 불변 컬렉션은 얼마나 많은 변수가 컬렉션을 참조하는가와 관계없이 컬렉션을 절대 바꿀 수 없다.

###### 컬렉션 사용하기

```scala
val fileLines = Source.fromFile("data.txt").getLines.toList()
val linesLongUpper = 
  fileLines.filter(l => l.length() > 10)
           .map(l => l.toUpperCase())

val linesLongUpperUsingInfix =
  fileLines filter (_.length > 10) map(_.toUpperCase())
```

* 스칼라의 컬렉션 동작은 스트림 API와 비슷하다.
* 언더스코어는 인수로 대치된다. 즉 `_.length()`는 `l.length()`로 해석할 수 있다.
  * `filter`와 `map`으로 전달된 함수에서 언더스코어는 처리되는 행으로 바운드된다.
* `par`이라는 메서드로 자바의 스트림의 `parallel`과 비슷하게 병렬 실행을 할 수 있다.

```scala
val linesLongUpper = fileLines.par filter (_.length() > 10) map(_.toUpperCase())
```

###### 튜플
* 스칼라는 튜플 축약어, 즉 간단한 문법으로 튜플을 만들 수 있는 기능을 제공하며 임의 크기(최대 23개 요소를 그룹화 가능)의 튜플을 제공한다.
  * `_1` 등의 접근자로 튜플의 요소에 접근할 수 있다.


```scala
val raoul = ("R", "+44 2342342")
val book = (2018, "modern java in action", "manning")

println(raoul._1) // R
println(book._3) // manning
```


###### 스트림
* 리스트, 집합, 맵, 튜플은 적극적으로(즉, 즉시) 평가되었다.
* 스칼라에서도 자바와 마찬가지로 게으르게 평가되는 스트림이라는 자료구조를 제공한다.
  * 이전 요소가 접근할 수 있도록 기존 계산값을 기억한다.
  * 인덱스를 제공해 리스트처럼 요소에 접근한다. 따라서 자바의 스트림에 비해 메모리 효율성은 조금 떨어진다. 이전 요소를 참조하려면 요소를 기억(캐시)해야 하기 때문이다.

###### 옵션
* 스칼라의 `Option`은 자바의 `Optional`과 같은 기능을 제공한다.

```scala
def getCarInsuranceName(person: Option[Person], minAge: Int) =
  person.filter(_.getAge() >= minAge)
        .flatMap(_.getCar) // 인수가 없는 메서드 호출 시 괄호 생략 가능
        .flatMap(_.getInsurance)
        .map(_.getName)
        .getOrElse("Unknown)
```


#### 20.2 함수
* 스칼라의 함수는 어떤 작업을 수행하는 일련의 명령어 그룹이다.
  * 함수 형식: 자바 함수 디스크립터의 개념을 표현하는 편의 문법(즉, 함수형 인터페이스에 선언된 추상 메서드의 시그니처를 표현하는 개념)이다.
  * 익명 함수: 자바의 람다 표현식과 달리 비지역 변수 기록에 제한을 받지 않는다.
  * 커링 지원: 여러 인수를 받는 함수를 일부 인수를 받는 여러 함수로 분리하는 기법이다.

##### 20.2.1 스칼라의 일급 함수
* 스칼라의 함수는 일급값으로 함수를 인수로 전달하거나 결과로 반환하거나 변수에 저장할 수 있다.

```scala
def isJavaMentioned(tweet: String) : Boolean = tweet.contains("Java")
def isShortTweet(tweet: String) : Boolean = tweet.length() < 20

tweets.filter(isJavaMentioned).foreach(println)

// filter 의 시그니처
def filter[T](p: (T) => Boolean): List[T]
```


* `(T) => Boolean`은 T라는 형식의 객체를 받아 Boolean을 반환함을 의미한다.


##### 20.2.2 익명 함수와 클로저
* 스칼라도 익명 함수의 개념을 지원하며 람다 표현식과 비슷한 문법을 제공한다.

```scala
val isLongTweet : String => Boolean = 
  new Function1[String, Boolean] {
    def apply(tweet: String): Boolean = tweet.length() > 60
  }
}
```

* 자바에서는 람다 표현식을 사용할 수 있도록 `Predicate`, `Function`, `Consumer` 등의 내장 함수형 인터페이스를 제공했다. 스칼라는 트레이트를 지원한다.
  * `Fuction0`(인수가 없으며 결과를 반환)에서 `Function22`(22개의 인수를 받음)를 제공한다.
  * 보통 함수를 호출하는 것처럼 `apply` 메서드를 호출할 수 있다.

###### 클로저
* 클로저란 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스를 가리킨다.
  * 자바의 람다 표현식에서는 람다가 정의된 메서드의 지역 변수를 고칠 수 없다. 이들 변수는 암시적으로 `final`로 취급된다. 즉, 람다는 **변수가 아닌 값을 닫는다**.
  * 스칼라의 익명 함수는 값이 아니라 변수를 캡처할 수 있다.

```scala
def main(args: Array[String]) {
  var count = 0
  val inc = () => count += 1
  inc()
  println(count) // 1 출력
}
```


##### 20.2.3 커링
* 여러 인수를 가진 함수를 커링으로 일반화할 수 있다. 즉, 여러 인수를 받는 함수를 인수의 일부를 받는 여러 함수로 분할할 수 있다.

```scala
def multiply(x: Int, y: Int) = x * y
val r = multiply(2, 10)

def multiplyCurry(x: Int)(y: Int) = x * y // 커리된 함수 정의
val r = multiplyCurry(2)(10) // 커리된 함수 호출
```

* 파라미터 하나로 `multiplyCurry`를 처음 호출하면 다른 파라미터를 인수로 받는 다른 함수를 반환하며 이를 캡처값(처음의 파라미터 하나)에 곱한다.
  * 모든 인수를 사용하지 않았으므로 이 상황을 함수가 부분 적용되었다고 표현한다.

```scala
val multiplyByTwo : Int => Int = multiplyCurry(2)
val r = multiplyByTwo(10) // 20
```

* 자바와 달리 스칼라에서는 함수가 여러 커리된 인수 리스트를 포함하고 있음을 가리키는 함수 정의 문법을 제공하므로 커리된 함수를 직접 제공할 필요가 없다.

#### 20.3 클래스와 트레이트

##### 20.3.1 간결성을 제공하는 스칼라의 클래스
* 스칼라는 완전한 객체지향 언어이므로 클래스를 만들고 객체로 인스턴스화할 수 있다.

```scala
class Hello {
  def sayThankYou() {
    println("Thanks for reading our book")
  }
}
val h = new Hello()
h.sayThankYou()
```

###### 게터와 세터
* 스칼라에서는 생성자, 게터, 세터가 암시적으로 생성된다.

##### 20.3.2 스칼라 트레이트와 자바 인터페이스
* 스칼라의 트레이트는 자바의 인터페이스를 대체한다. 트레이트로 추상 메서드와 기본 구현을 가진 메서드 두 가지를 모두 정의할 수 있다.
  * 다중 상속을 지원하므로 자바의 인터페이스와 디폴트 메서드 기능이 합쳐진 것으로 이해할 수 있다.

```scala
trait Sized {
  var size : Int = 0
  def isEmpty() = size == 0
}

class Empty extends Sized

println(new Empty().isEmpty()) // true 출력
```

* 객체 트레이트는 인스턴스화 과정에서도 조합알 수 있다.

```scala
class Box
val b1 = new Box() with Sized // 객체 인스턴스화할 때 트레이트를 조합함
println(b1.isEmpty()) // true 출력
val b2 = new Box()
b2.isEmpty() // 컴파일 에러
```

* 같은 시그니처를 가진 메서드나 같은 이름을 가진 필드를 정의하는 트레이트를 다중 상속하는 경우 자바와 비슷한 제한을 둔다.
