---
title: "Kotlin in Action 2장; 코틀린 기초"
categories:
  - Language
tags:
  - kotlin
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 2. 코틀린 기초

* 함수, 변수, 클래스, `enum`, 프로퍼티를 선언하는 방법
* 제어 구조
* 스마트 캐스트
* 예외 던지기와 예외 잡기



#### 2.1 기본 요소: 함수와 변수

##### Hello, World!

```kotlin
fun main(args: Array<String) {
  println("Hello, world!")
}
```

* 함수 선언 시 `fun` 키워드를 사용한다.
* 파라미터 이름 뒤에 그 파라미터의 타입을 쓴다.
* 함수를 최상위 수준에 정의할 수 있다.
* 배열도 일반적인 클래스와 마찬가지다.

##### 함수

```kotlin
fun max(a: Int, b: Int): Int {
  return if (a > b) a else b
}
```

* 함수 선언은 `fun` 키워드로 시작하고, 그 다음 함수 이름이 온다.
* 이름 뒤에는 괄호 안에 파라미터 목록이 온다. 함수 반환 타입은 파라미터 목록의 닫는 괄호 다음에 오는데, 괄호와 반환 타입 사이를 콜론(`:`)으로 구분해야 한다.

> 코틀린에서 `if`는 식이지 문이 아니다. 식은 값을 만들어 내며 다른 식의 하위 요소로 계산에 참여할 수 있는 반면 문은 자신을 둘러싸고 있는 가장 안쪽 블록의 최상의 요소로 존재하며 아무런 값을 만들어 내지 않는다는 차이가 있다. 코틀린에서는 루프를 제외한 대부분의 구조가 식이다. 반면 대입문은 자바에서는 식이었으나 코틀린에서는 문이 됐다.

* 식 하나로만 이루어진 함수의 경우 더 간결하게 함수를 표현할 수 있다.
  * 본문이 중괄호로 둘러싸인 함수를 블록이 본문인 함수라 부르고 등호와 식으로 이뤄진 함수를 식이 본문인 함수라고 부른다.

```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

> 인텔리제이는 이 두 방식의 함수를 서로 변환하는 메뉴가 있다. *Convert to expression body*와 *Convert to block body*로 가능하다.

* 반환 타입의 생략도 가능하다.
  * **식이 본문인 함수**의 경우 굳이 사용자가 반환 타입을 적지 않아도 컴파일러가 함수 본문 식을 분석해서 식의 결과 타입을 함수 반환 타입으로 정해 준다.
  * 이렇게 컴파일러가 타입을 분석해 프로그래머 대신 프로그램 구성 요소의 타입을 정해 주는 기능을 **타입 추론**이라 한다.

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

* 블록이 본문인 함수가 값을 반환한다면 반드시 타입을 지정하고 `return` 문을 사용해 반환 값을 명시해야 한다.

##### 변수

* 타입 지정을 생략하는 경우가 많다. 타입으로 변수 선언을 시작하면 타입을 생략할 경우 식과 변수 선언을 구별할 수 없기 때문이다.
* 변수 이름 뒤에 타입을 명시하거나 생략하게 허용한다.

```kotlin
val question = "dinner menu?"
val answer = 42
val age: Int = 24
```

* 타입을 지정하지 않으면 컴파일러가 초기화 식을 분석해서 초기화 식의 타입을 변수 타입으로 지정한다. 초기화 식을 사용하지 않고 변수를 선언하려면 변수 타입을 반드시 명시해야 한다.
* 변수 선언 시 사용하는 키워드는 다음과 같은 2가지가 있다.
  * `val`: 변경 불가능한 참조를 저장하는 변수다. 초기화하고 나면 재대입이 불가능하다.
  * `var`: 변경 가능한 참조다. 자바의 일반 변수에 해당한다. 
* 기본적으로는 모든 변수를 `val` 키워드를 사용해 불변 변수로 선언하고, 나중에 꼭 필요할 때만 `var`로 변경하는 게 낫다. 변경 불가능한 참조와 변경 불가능한 객체를 부수 효과가 없는 함수와 조합해 사용하면 코드가 함수형 코드에 가까워진다.
* `val` 변수는 블록을 실행할 때 정확히 한 번만 초기화돼야 한다. 하지만 어떤 블록이 실행될 때 오직 한 초기화 문장만 실행됨을 컴파일러가 확인할 수 있다면 조건에 따라 `val` 값을 다른 여러 값으로 초기화할 수 있다.

```kotlin
val message: String
if (operation()) {
  message = "success"
  // ...
} else {
  message = "fail"
}
```

* `val` 참조 자체는 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.

```kotlin
val languages = arrayListOf("java")
languages.add("c")
```

* `var` 키워드를 사용하면 변수의 값을 변경할 수 있지만 변수의 타입은 고정돼 바뀌지 않는다.
  * 컴파일러는 변수 선언 시점의 초기화 식으로부터 변수의 타입을 추론하며 변수 선언 이후 재대입이 이뤄질 때는 이미 추론한 변수의 타입을 염두에 두고 대입문의 타입을 검사한다.

##### 더 쉽게 문자열 형식 지정: 문자열 템플릿

```kotlin
fun main(args: Array<String>) {
  val name = if (args.size > 0) args[0] else "kotlin"
  println("Hello, $name!")
}
```

* 변수를 문자열 안에 사용할 수 있다. 문자열 리터럴의 필요한 곳에 변수를 넣되 변수 앞에 `$`를 추가해야 한다.
* `$` 문자를 문자열에 넣고 싶으면 `println("\$x")`와 같이 `\`를 사용해 이스케이프시켜야 한다.
* 복잡한 식도 중괄호로 둘러싸서 문자열 템플릿 안에 넣을 수 있다.

```kotlin
fun main(args: Array<String>) {
  if (args.size > 0) {
    println("Hello, ${args[0]}!")
  }
}
```

> 문자열 템플릿 안에 `$`로 변수를 지정할 때 변수명 뒤에 한글을 붙여서 사용하면 코틀린 컴파일러는 영문자와 한글을 한꺼번에 식별자를 인식해서 오류를 발생시킨다. 변수 이름을 중괄호로 감싸면 해결 가능하다.

* 중괄호로 둘러싼 식 안에서 큰 따옴표를 사용할 수도 있다.



#### 2.2 클래스와 프로퍼티

```java
/* java */
public class Person {
  private final String name;
  
  public Person(String name) {
    this.name = name;
  }
  
  public String getName() {
    return name;
  }
}
```

* 자바에서는 생성자 본문에 같은 코드가 반복적으로 들어가는 경우가 많다. 코틀린에서는 필드 대입 로직을 훨씬 더 적은 코드로 작성할 수 있다.

```kotlin
class Person(val name: String)
```

* 이런 유형의 클래스(코드가 없이 데이터만 저장하는 클래스)를 **값 객체**라고 부른다.
* 코틀린의 기본 가시성은 `public`이므로 변경자를 생략해도 된다.

##### 프로퍼티

* 클래스의 목적은 데이터를 캡슐화하고 캡슐화한 뎅터를 다루는 코드를 한 주체 안에 가두는 것이다. 자바에서는 데이터를 필드에 저장하며 멤버 필드의 가시성은 보통 비공개(`private`)다.  클래스느느 자신을 사용하는 클라이언트가 데이터에 접근하는 통로로 쓸 수 있는 접근자 메소드를 제공한다.
  * 자바에서는 필드와 접근자를 한데 묶어 프로퍼티라고 부르며, 프로퍼티 개념을 활용하는 프레임워크가 많다.
* 코틀린은 프로퍼티를 언어 기본 기능으로 제공하며 자바의 필드와 접근자 메소드를 완전히 대신한다.
  * 클래스에서 프로퍼티를 선언할 때는 `val`이나 `var`를 사용한다.

```kotlin
class Person (
  val name: String,
  var isMarried: Boolean
)
```

* 기본적으로 코틀린에서 프로퍼티를 선언하는 방식은 프로퍼티와 관련 있는 접근자를 선언하는 것이다.
  * 코틀린은 값을 저장하기 위한 비공개 필드와 세터, 게터로 이루어진 간단한 디폴트 접근자 구현을 제공한다.
* 위의 자바로 작성한 클래스와 코틀린으로 작성한 클래스는 동일하게 구현된 클래스이다.
  * 어느 쪽을 사용해도 코드를 바꿀 필요가 없다.
  * 게터와 세터의 이름을 정하는 규칙에는 예외가 있는데, 이름이 `is`로 시작하는 프로퍼티의 게터에는 원래 이름을 그대로 사용한다. 세터에는 `is`를 `set`으로 바꾼 이름을 사용한다.

> 자바에서 선언한 클래스에 대해 코틀린 문법을 사용해도 된다.

##### 커스텀 접근자

* 프로퍼티의 접근자를 직접 작성하기
  * `isSquare` 프로퍼티에는 자체 값을 저장하는 필드가 없이 자체 구현을 제공하는 게터만 존재한다.
  * 게터가 프로퍼티 값을 매번 다시 계산한다. 이를 자바에서 사용하려면 `isSquare` 메소드를 호출하면 된다.

```kotlin
class Rectangle(val height: Int, var width: Int) {
  val isSquare: Boolean
    get() {
      return height == width
    }
}
```

* 파라미터가 없는 함수를 정의하는 방식과 커스텀 게터를 정의하는 방식 모두 비슷하다. 구현이나 성능상 차이는 없다.
  * 일반적으로 클래스의 특성을 정의하고 싶다면 프로퍼티로 그 특성을 정의해야 한다.

##### 코틀린 소스코드 구조: 디렉터리와 패키지

* 코틀린에도 자바와 비슷한 개념의 패키지가 있으며 모든 코틀린 파일의 맨 앞에 `package` 문을 넣을 수 있다. 그러면 그 파일에 있는 모든 선언이 해당 패키지에 들어간다.
* 같은 패키지에 속해 있다면 다른 파일에서 정의한 선언일지라도 직접 사용할 수 있다. 다른 패키지의 경우 임포트를 통해 불러온다.

```kotlin
package geometry.shapes

import java.util.Random

// ...
```

* 코틀린에서는 클래스 임포트와 함수 임포트에 차이가 없으며 모든 선언을 `import` 키워드로 가져올 수 있다.
* `.*`로 패키지 안의 모든 선언을 임포트할 수 있다. 클래스뿐 아니라 최상위에 정의된 함수나 프로퍼티까지 불러온다.
* 코틀린에서는 여러 클래스를 한 파일에 넣을 수 있고 파일의 이름도 마음대로 정할 수 있다.
  * 디스크상의 어떤 디렉터리에 소스코드 파일을 위치시키든 관계없다. 원하는 대로 소스코드를 구성할 수 있다.
  * 하지만 대부분의 경우 자바-코틀린 간 마이그레이션을 위해 자바와 같이 패키지별로 디렉터리를 구성하는 폇이 낫다.



#### 2.3 선택 표현과 처리: enum과 when

##### enum 클래스 정의

```kotlin
enum class Color {
  RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

* `enum`은 소프트 키워드로 `class` 앞에 있을 때는 특별한 의미를 지니지만 다른 곳에서는 이름에 사용할 수 있다.
* `enum` 클래스 안에도 프로퍼티나 메소드를 정의할 수 있다.

```kotlin
enum class Color (
  val r: Int, val g: Int, val b: Int
) {
  RED(255, 0, 0), ORANGE(255, 165, 0),
  YELLOW(255, 255, 0), GREEN(0, 255, 0), BLUE(0, 0, 255),
  INDIGO(75, 0, 130), VIOLET(238, 130, 238);
  
  fun rgb() = (r * 256 + g) * 256 + b
}
```

* 일반적인 클래스와 마찬가지로 생성자와 프로퍼티를 선언한다.
* 각 `enum` 상수를 정의할 때는 그 상수에 해당하는 프로퍼티 값을 지정해야만 한다. 
  * 메소드를 정의하는 경우 반드시 `enum` 상수 목록과 메소드 정의 사이에 세미콜론을 넣어야 한다.

##### when으로 enum 클래스 다루기

* 자바의 `switch` 에 해당하는 코틀린 구성 요소는 `when`이다. 
  * 값을 만들어내는 식이므로 식이 본문인 함수에 `when`을 바로 사용할 수 있다.

```kotlin
fun getMnemonic(color: Color) =
	when (color) {
    Color.RED -> "Richard"
    Color.ORANGE -> "Of"
    Color.YELLOW -> "York"
    Color.GREEN -> "Gave"
    Color.BLUE -> "Battle"
    Color.INDIGO -> "In"
    Color.VIOLET -> "Vain"
  }
```

* 자바와 달리 각 분기의 끝에 `break`를 넣지 않아도 된다. 성공적으로 매치되는 분기를 찾으면 그 분기를 실행한다.
* 한 분기 안에서 여러 값을 매치 패턴으로 사용할 수 있다. 그런 경우 값 사이를 콤마(`,`)로 분리한다.

```kotlin
fun getMnemonic(color: Color) =
	when (color) {
    Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
    Color.GREEN -> "neutral"
    Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
  }
/* ------------------------------------------------- */

/* 상수 값을 임포트하면 더 간단하게 사용 가능하다.*/
import ch02.colors.Color
import ch02.colors.Color.*
fun getMnemonic(color: Color) =
	when (color) {
    RED, ORANGE, YELLOW -> "warm"
    GREEN -> "neutral"
    BLUE, INDIGO, VIOLET -> "cold"
  }
```

##### when과 임의의 객체를 함께 사용

* `when`의 분기 조건은 임의의 객체를 허용한다.

````kotlin
fun mix(c1: Color, c2: Color) =
   when (setOf(c1, c2)) {
     setOf(RED, YELLOW) -> ORANGE
     setOf(YELLOW, BLUE) -> GREEN
     setOf(BLUE, VIOLET) -> INDIGO
     else -> throw Exception("dirty color")
   }
````

* 코틀린 표준 라이브러리에는 인자로 전달받은 여러 객체를 그 객체들을 포함하는 `Set` 객체로 만드는 `setOf`라는 함수가 있다. 
* `when` 식은 인자 값과 매치하는 조건 값을 찾을 때까지 각 분기를 검사한다.
  * 여기서 `setOf(c1, c2)`와 분기 조건에 있는 객체 사이를 매치할 때 동등성을 사용한다. 
  * 모든 분기 식에서 만족하는 조건을 찾을 수 없다면 `else` 분기의 문장을 계산한다.
* `when`의 분기 조건 부분에 식을 넣을 수 있어 코드를 더 간결하게 작성할 수 있다. 

##### 인자 없는 when 사용

* 인자가 없는 `when` 을 사용하면 불필요한 객체 생성을 막을 수 있다.
  * 아무 인자가 없으려면 각 분기의 조건이 불리언 결과를 계산하는 식이어야 한다.
  * 가독성은 떨어지지만 성능은 향상시킬 수 있다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) = 
   when {
     (c1 == RED && c2 == YELLOW) ||
     (c1 == YELLOW && c2 == RED) -> ORANGE
     (c1 == YELLOW && c2 == BLUE) ||
     (c1 == BLUE && c2 == YELLOW) -> GREEN
     (c1 == BLUE && c2 == VIOLET) ||
     (c1 == VIOLET && c2 == BLUE) -> INDIGO
     else -> throw Exception("dirty color")
   }
```

##### 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

* 간단한 산술식을 계산하는 함수를 예시로 만들어 본다.
  * 식을 트리 구조로 저장하자. 노드는 합계(`Sum`)나 수(`Num`) 중 하나다. 수(`Num`)는 항상 말단 노드지만 합계(`Sum`)는 자식이 둘 있는 중간 노드다. `Sum` 노드의 자식은 덧셈의 두 인자이다. 

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

* 식을 위한 `Expr` 인터페이스가 있고, `Sum`과 `Num` 클래스는 인터페이스를 구현한다. 인터페이스는 아무 메소드도 선언하지 않으며 단지 공통 타입 역할만 수행한다. 클래스가 구현하는 인터페이스를 지정하기 위해서 콜론(`:`) 뒤에 인터페이스 이름을 사용한다.

* (1 + 2) + 4라는 식은 `Sum(Sum(Num(1), Num(2)), Num(4))`로 표현될 수 있다. 식의 값은 어떻게 계산할까?

  * `Expr` 인터페이스에는 두 가지 구현 클래스가 존재한다. 따라서 식을 평가하려면 두 가지 경우를 고려해야 한다.

  1. 어떤 식이 수라면 그 값을 반환한다.
  2. 어떤 식이 합계라면 좌항과 우항의 값을 계산한 다음 합한 값을 반환한다.

* `if`를 써서 자바 스타일로 작성하면 다음과 같다.

```kotlin
fun eval(e: Expr) : Int {
  if (e is Num) { // e를 Num으로 해석
    val n = e as Num
    return n.value
  }
  if (e is Sum) { // e를 Sum으로 해석
    return eval(e.right) + eval(e.left)
  }
  throw IllegalArgumentException("unknown expression")
}
```

* 코틀린에서는 `is`를 사용해 변수 타입을 검사한다.
  * 프로그래머 대신 컴파일러가 캐스팅을 해 주므로 검사하고 나면 굳이 변수를 원하는 타입으로 캐스팅하지 않아도 처음부터 변수가 원하는 타입으로 선언된 것처럼 사용할 수 있다.
  * 이를 **스마트 캐스트**라고 부른다.
* 스마트 캐스트는 `is` 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다.
  * 클래스의 프로퍼티에 대해 사용한다면 그 프로퍼티는 반드시 `val`이어야 한다.
  * 원하는 타입으로 명시적으로 캐스팅하려면 `as` 키워드를 사용한다.

##### 리팩토링: if를 when으로 변경

* `if`가 값을 만들어내므로 3항 연산자가 따로 없다. 이런 특성을 이용해 `if`식을 본문으로 사용해서 더 간단하게 만들 수 있다.

```kotlin
fun eval(e: Expr) : Int =
   if (e is Num) {
     e.value
   } else if (e is Sum) {
     eval (e.right) + eval(e.left)
   } else {
     throw IllegalArgumentException("unknown expression")
   }
```

* `if`의 분기에 식이 하나밖에 없다면 중괄호를 생략해도 된다. 분기에 블록을 사용하는 경우 그 블록의 마지막 식이 그 분기의 결과 값이다.

```kotlin
fun eval(e: Expr) : Int =
   when(e) {
     is Num -> e.value
     is Sum -> eval(e.right) + eval(e.left)
     else -> throw IllegalArgumentException("unknown expression")
   }
```

* 받은 값의 타입을 검사할 때에도 `when` 분기를 사용할 수 있다. 이 경우에도 역시 타입을 검사하고 나면 스마트 캐스트가 이뤄진다.

##### if와 when의 분기에서 블록 사용

* 각 분기에서 수행해야 하는 로직이 복잡해지면 분기 본문에 블록을 사용할 수 있다.
* `if`나 `when` 모두 분기에 블록을 사용할 수 있고, 블록의 마지막 문장이 블록 전체의 결과가 된다.

```kotlin
fun evalWithLogging(e: Expr) : Int =
   when(e) {
     is Num -> {
       println("num: ${e.value}")
       e.value
     } 
     is Sum -> {
       val left = evalWithLogging(e.left)
       val right = evalWithLoggin(e.right)
       println("sum: $left + $right")
       left + right
     } else -> throw IllegalArgumentException("unknown expression")
   }
```

* 블록의 마지막 식이 블록의 결과라는 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립한다. 이 규칙은 함수에 대해서는 성립하지 않는다.
  * 식이 본문인 함수는 블록을 본문으로 가질 수 없고 블록이 본문인 함수는 내부에 `return` 문이 반드시 있어야 한다.



#### 2.4 대상을 이터레이션: while과 for 루프

##### 2.4.1 while 루프

* `while`과 `do-while` 루프가 있는데, 문법은 자바와 같다.

##### 2.4.2 수에 대한 이터레이션: 범위와 수열

* 코틀린에서는 `for` 루프 대신 범위(`range`)를 사용한다.
  * `..` 연산자로 시작 값과 끝 값을 연결해서 범위를 만든다. 코틀린의 범위는 폐구간 또는 양끝을 포함하는 구간이다.
* 정수 범위로 수행할 수 있는 가장 단순한 작업은 범위에 속한 모든 값에 대한 이터레이션이다. 이런 식으로 범위에 속한 값을 일정한 순서로 이터레이션하는 경우를 수열(`progression`)이라고 부른다.

```kotlin
fun fizzBuzz(i: Int) = when {
  i % 15 == 0 -> "FizzBuzz"
  i % 3 == 0 -> "Fizz"
  i % 5 == 0 -> "Buzz"
  else -> "$i"
}
```
```kotlin
for(i in 1..100) { // 1부터 100까지의 정수에 대해 이터레이션
  print(fizzBuzz(i))
}
```

```kotlin
fun (i in 100 downTo 1 step 2) {
  print(fizzBuzz(i))
}
```

* 여기서는 증가 값 `step`을 갖는 수열에 대해 이터레이션한다. 증가 값을 사용하면 수를 건너 뛸 수 있다. 음수로 만들면 역방향 수열을 만들 수 있다.
  * 위의 예제에서 `100 downTo 1`은 역방향 수열을 만든다. 그 뒤에 `step 2`를 붙이면 증가 값의 절댓값이 2로 바뀐다.
* 반만 닫힌 범위롤 만들고 싶은 경우 `until`을 사용한다.

##### 맵에 대한 이터레이션

```kotlin
val binaryReps = TreeMap<Char, String>()

for (c in 'A'..'F') {
  val binary = Integer.toBinaryString(c.toInt())
  binaryReps[c] = binary
}

for ((letter, binary) in binaryReps) {
  println("$letter = $binary")
}
```

* `..` 연산자를 숫자뿐 아니라 문자 타입의 값에도 적용할 수 있다.
  * `for` 루프를 사용해 이터레이션하려는 컬렉션의 원소를 풀 수도 있다. 
* 맵을 값을 가져오거나 키에 해당하는 값을 쉽게 설정 가능하다.
* 맵에 사용했던 위와 같은 구조를 컬렉션에도 활용할 수 있다. 구조 분해 구문을 사용하면 원소의 현재 인덱스를 유지하면서 컬렉션을 이터레이션할 수 있다.

```kotlin
val list = arrayListOf("10", "11", "1001")
for ((index, element) in list.withIndex()) {
  println("$index: $element")
}
```

##### in으로 컬렉션이나 범위의 원소 검사

* `in` 연산자를 사용해 어떤 값이 범위에 속하는지 검사할 수 있다. 반대로 `!in`을 사용하면 어떤 값이 범위에 속하지 않는지 검사할 수 있다.

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

* `when` 식에서의 활용도 가능하다.

```kotlin
fun recognize(c: Char) = when(c) {
  in '0'..'9' -> "digit"
  in 'a'..'z', in 'A'..'Z' -> "letter"
  else -> "??"
}
```

* 범위는 문자에만 국한되지 않고 비교 가능한 클래스라면 그 클래스의 인스턴스 객체를 사용해 범위를 만들 수 있다.
  * `Comparable` 을 사용하는 범위의 경우 그 범위 내의 모든 객체를 항상 이터레이션하지는 못한다.
  * 하지만 `in` 연산자를 사용하면 값이 범위 안에 속하는지 항상 결정할 수 있다.

```kotlin
println("kotlin" in "java".."scala") // "java" <= "kotlin" && "kotlin" <= "scala"
```

* 컬렉션에서도 사용 가능하다.



#### 2.5 코틀린의 예외 처리

* 코틀린의 예외 처리는 자바와 비슷하다.
  * 함수는 정상적으로 종료할 수 있지만 오류가 발생하면 던질 수 있고, 함수를 호출하는 쪽에서는 예외를 잡아 처리할 수 있다. 발생한 예외를 함수 호출 단에서 처리하지 않으면 스택을 거슬러 올라가며 예외를 처리하는 부분이 나올 때까지 다시 던진다.

```kotlin
if (percentage !in 0..100) {
  thrwo IllegalArgumentException("percentage value must be between 0 and 100")
}
```

* 예외 인스턴스를 만들 때도 `new`를 붙일 필요가 없다. 코틀린의 `throw`는 식이므로 다른 식에 포함될 수 있다.

##### 2.5.1 try, catch, finally

```kotlin
fun readNumber(reader: BufferedReader): Int? {
  try {
    val line = reader.readLine()
    return Integer.parseInt(line)
  }
  catch (e: NumberFormatException) {
    return null
  }
  finally {
    reader.close()
  }
}
```

* 자바 코드와 가장 큰 차이는 `throws` 절이 코드에 없다.

  * 자바에서는 함수를 작성할 때 함수 선언 뒤에 `throws IOException`을 붙여야 한다. 체크 예외를 명시적으로 처리해야 하기 때문이다. 어떤 함수가 던질 가능성이 있는 예외나 그 함수가 호출한 다른 함수에서 발생할 수 있는 예외를 모두 처리해야 하며, 처리하지 않은 예외는 `throws` 절에 명시해야 한다.

  * 코틀린은 체크 예외와 언체크 예외를 구별하지 않는다. 실제 자바 프로그래머들은 의미 없이 예외를 다시 던지거나 예외를 잡되 처리하지는 않고 그냥 무시하는 코드를 작성하는 경우가 흔하므로 그러한 방식을 고려해 예외를 설계했다.

##### try를 식으로 사용

```kotlin
fun readNumber(reader: BufferedReader): Int? {
  val number = try {
    Integer.parseInt(reader.readLine())
  } catch (e: NumberFormatException) {
    return
  }
  println(number)
}
```

* `try` 키워드는 식이므로 값을 변수에 대입할 수 있다. 본문은 반드시 중괄호로 둘러싸야 한다. 다른 문장과 마찬가지로 `try`의 본문도 내부에 여러 문장이 있으면 마지막 식의 값이 전체 결과 값이다.
  * `catch` 이후에도 계속 진행하고 싶다면 `catch` 블록도 값을 만들어야 한다. 마찬가지로 마지막 식이 블록 전체의 값이 된다.

```kotlin
fun readNumber(reader: BufferedReader): Int? {
  val number = try {
    Integer.parseInt(reader.readLine())
  } catch (e: NumberFormatException) {
    null
  }
  println(number)
}
```

* `try` 코드 블록의 실행이 정상적으로 끝나면 그 블록의 마지막 식의 값이 결과다. 예외가 발생하고 잡히면 그 예외에 해당하는  `catch` 블록의 값이 결과다.
