---
title: "Kotlin in Action 6장; 코틀린 타입 시스템"
categories:
  - Language
tags:
  - kotlin
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 6. 코틀린 타입 시스템

#### 6.1 널 가능성

* 가능한 `null`을 컴파일 시점에서 알 수 있도록 해서 예외의 가능성을 줄인다.

##### 널이 될 수 있는 타입

* 널이 될 수 있는 타입을 명시적으로 지원한다.
  * `null`이 될 수 없는 인자를 가지는 함수에 `null` 인자를 넘기면 컴파일 오류가 발생한다.
* `null`을 받을 수 있게 하려면 타입 이름 뒤에 물음표를 명시한다.
  * 수행할 수 있는 연산이 제한된다.
  * 널이 될 수 있는 값을 널이 될 수 없는 타입의 변수에 대입할 수 없고, 널이 될 수 있는 타입의 값을 널이 될 수 없는 타입의 파라미터를 받는 함수에 전달할 수 없다.
  * `null`과 비교해서 `null`이 아님이 확시한 영역에서는 널이 될 수 없는 타입의 값처럼 사용 가능하다.

##### 타입의 의미

* 타입은 어떤 값들이 가능한지와 그 타입에 대해 수행할 수 있는 연산의 종류를 결정하지만 자바의 타입 시스템은 널을 제대로 다루지 못한다.
* 코틀린에서는 널이 될 수 있는 타입과 없는 타입을 구분하고 모든 검사는 컴파일 시점에 수행되어 널이 될 수 있는 타입을 처리하는 데에 별도의 비용이 들지 않는다.

##### 안전한 호출 연산자: ?.

* `?.`은 `null` 검사와 메소드 호출을 한 번의 연산으로 수행한다.
  * 호출하려는 값이 `null`이 아니면 일반 메소드 호출처럼 작동한다.
  * 호출하려는 값이 `null`이면 호출은 무시되고 `null`이 결과 값이 된다.
* 메소드 호출뿐 아니라 프로퍼티를 읽거나 쓸 때도 안전한 호출을 사용할 수 있다.

##### 엘비스 연산자: ?:

* 엘비스 연산자 `?:`는 `null` 대신 사용할 디폴트 값을 저장할 때 편리하게 사용할 수 있는 연산자이다. 
  * 좌항 값이 널이 아니면 좌항 값을 결과로 하고 좌항 값이 널이면 우항 값을 결과로 한다.
* 코틀린에서는 `return`이나 `throw` 등의 연산도 식이다.
  * 엘비스 연산자의 우항에 해당 연산을 넣을 수도 있다.

##### 안전한 캐스트: as?

* `as?` 연산자는 어떤 값을 지정한 타입으로 캐스트한다.
  * 대상 타입으로 변환할 수 없으면 `null`을 반환한다.
* `as?`와 `?:`을 함께 사용하면 파라미터로 받은 값이 원하는 타입인지 쉽게 검사하고 캐스트한 뒤에 타입이 맞지 않으면 `false`를 반환할 수 있다.

##### 널 아님 단언: !!

* 어떤 값이든 널이 될 수 없는 타입으로 바꿀 수 있다.
  * 실제 널에 대해 적용하면 NPE가 발생한다.
* 어떤 값이 널이었는지 확실히 하기 위해 여러 `!!` 단언문을 한 줄에 함께 쓰면 어떤 식에서 예외가 발생했는지 알 수 없음에 주의해야 한다.

##### let 함수

* 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘길 때에 쓸 수 있다.
* `let` 함수를 안전한 호출 연산자와 함께 사용하면 원하는 식을 평가해서 결과가 널인지 검사한 다음에 그 결과를 변수에 넣는 작업을 쉽게 처리할 수 있다.
  * 자신의 수신 객체를 인자로 전달받은 람다에게 넘긴다. 널이 될 수 있는 값에 대한 안전한 호출 구문을 사용해 `let`을 호출하되 널이 될 수 없는 타입을 인자로 받는 람다를 `let`에 전달한다.
  * 수신 객체가 `null`이 아니면 람다 안에서 그 객체는 널이 아니고 `null`이라면 아무 일도 일어나지 않는다. 즉, 수신 객체가 널이 아닌 경우에만 람다를 실행한다.

```kotlin
email?.let { email -> sendEmailTo(email) } // email이 null이 아닌 경우에만 sendEmailTo
```

##### 나중에 초기화할 프로퍼티

* 코틀린에서는 생성자에서 모든 프로퍼티를 초기화해야 한다. 게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 그 프로퍼티를 초기화해야 한다.
  * 초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용할 수밖에 없다.
* `lateinit` 변경자를 붙이면 프로퍼티를 나중에 초기화할 수 있다.
  * 나중에 초기화하는 프로퍼티는 항상 `var` 이어야 한다.
  * `val` 프로퍼티는 `final` 필드로 컴파일되며 생성자 안에서 반드시 초기화해야 하기 때문이다.

##### 널이 될 수 있는 타입 확장

* 널이 될 수 있는 타입에 대한 확장 함수를 정의하면 해당 메소드가 알아서 널을 처리해 준다.
  * 이런 처리는 확장 함수에서만 가능하다. 일반 멤버 호출은 객체 인스턴스를 통해서 디스패치되므로 인스턴스가 널인지 여부를 검사하지 않는다.
* 코틀린에서는 널이 될 수 있는 타입의 확장 함수 안에서는 `this`가 널이 될 수 있다.
  * `let` 함수도 널이 될 수 있는 타입의 값에 대해 호출할 수 있지만 `let`은 `this`가 널인지 검사하지 않는다. 널이 될 수 있는 타입의 값에 대해 안전한 호출을 사용하지 않고 `let`을 호출하면 람다의 인자는 널이 될 수 있는 타입으로 추론된다.

##### 타입 파라미터의 널 가능성

* 코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다. 널이 될 수 있는 타입을 포함하는 어떤 타입이라도 타입 파라미터를 대신할 수 있다.
  * 타입 파라미터 `T`를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라고 널이 될 수 있는 타입이다.
* 타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한을 지정해야 한다.

```kotlin
fun <T: Any> printHashCode(t: T) { // T는 널이 될 수 없는 타입이다.
  println(t.hashCode())
}
```

##### 널 가능성과 자바

* 자바 코드에 어노테이션으로 표시된 널 가능성 정보가 있다면 코틀린은 그 정보를 활용한다.
* 이런 널 가능성이 어노테이션이 소스 코드에 없는 경우 자바의 타입은 코틀린의 플랫폼 타입이 된다.

###### 플랫폼 타입

* 코틀린이 널 관련 정보를 알 수 없는 타입을 말하며 널이 될 수 있는 타입으로 처리해도 되고, 널이 될 수 없는 타입으로 처리해도 된다.
* 모든 연산에 대한 책임은 개발자에게 있다. 추가 검사가 없다면 널 관련 예외가 발생할 수 있는데, NPE보다 더 자세한 예외가 발생할 수 있다.
  * 코틀린 컴파일러는 공개 가시성인 코틀린 함수의 널이 아닌 타입인 파라미터와 수신 객체에 대한 널 검사를 추가해 준다. 따라서 공개 가시성 함수에 널 값을 사용하면 즉시 예외가 발생한다. 이런 파라미터 값 검사는 함수 호출 시점에 이뤄진다.
* 자바 코드에서 가져온 타입만 플랫폼 타입이 된다.

###### 상속

* 코틀린에서 자바 메소드를 오버라이드할 때 그 메소드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언해야 할지 결정해야 한다.
* 자바 클래스나 인터페이스를 코틀린에서 구현할 경우 널 가능성을 제대로 처리해야 한다.



#### 6.2 코틀린의 원시 타입

##### 원시 타입, Int, Boolean 등

* 코틀린은 원시 타입과 래퍼 타입을 구분하지 않으므로 항상 같은 타입을 사용한다.
* 실행 시점에 숫자 타입은 가능한 한 가장 효율적인 방식으로 표현된다. 대부분의 경우 코틀린의 `Int` 타입은 자바의 `int` 타입으로 컴파일된다.
  * 이런 컴파일이 불가능한 경우는 컬렉션과 같은 제네릭 클래스를 사용하는 경우뿐이다.
* `Int`와 같은 코틀린 타입에는 널 참조가 들어갈 수 없기 때문에 쉽게 그에 상응하는 자바 원시 타입으로 컴파일할 수 있다.

##### 널이 될 수 있는 원시 타입: Int?, Boolean? 등

* 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일된다.
  * 가령 널이 될 수 있는 `Int` 타입은 `java.lang.Integer` 로 저장된다.
* 제너릭 클래스의 경우 래퍼 타입을 사용한다.
  * 어떤 클래스의 타입 인자로 원시 타입을 넘기면 코틀린은 그 타입에 대한 박스 타입을 사용한다.
  * JVM은 타입 인자로 원시 타입을 허용하지 않아 제네릭 클래스는 항상 박스 타입을 사용해야 한다.
  * 원시 타입으로 이뤄진 대규모 컬렉션을 효율적으로 저장해야 한다면 원시 타입으로 이뤄진 효율적인 컬렉션을 제공하는 서드파티 라이브러리를 사용하거나 배열을 사용해야 한다.

##### 숫자 변환

* 코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다.
* 코틀린은 모든 원시 타입에 대한 변환 함수를 제공한다.
  * `toByte()`, `toShort()` 등
  * 양방향 변환 함수가 모두 제공된다.
* 코틀린은 개발자의 혼란을 피하기 위해 타입 변환을 명시하기로 결정했다.
  * 묵시적 변환은 이루어지지 않는다.
  * 타입을 명시적으로 변환해서 같은 타입의 값으로 만든 후 비교해야 한다.
* 숫자 리터럴을 사용할 때는 변환 함수를 호출할 필요가 없다.
* 산술 연산자는 적당한 타입의 값을 받아들일 수 있게 이미 오버로드돼 있다.
  * 자바와 똑같이 숫자 연산 시 오버플로우가 발생할 수 있다.

##### Any, Any?: 최상위 타입

* 코틀린에서는 `Any` 타입이 모든 **널이 될 수 없는** 타입의 조상 타입이다.
  * 원시 타입을 포함한 모든 타입의 조상 타입이다.
* 원시 타입 값을 `Any` 타입 변수에 대입하면 자동으로 값을 객체로 감싼다.
  * 자바 메소드에서 `Object`를 인자로 받거나 반환하면 코틀린에서는 `Any`로 그 타입을 취급한다.
  * 코틀린 함수가 `Any`를 사용하면 자바 바이트코드의 `Object`로 컴파일된다.
* 널을 포함하는 모든 값을 대입할 변수를 선언하려면 `Any?` 타입을 사용해야 한다.

##### Unit 타입: 코틀린의 void

* 코틀린 `Unit` 타입은 자바 `void`와 같은 기능을 한다.
* 코틀린의 `Unit`과 자바 `void`의 차이는 무엇일까?
  * `Unit`은 모든 기능을 갖는 일반적인 타입이며 타입 인자로 쓸 수 있다.
  * `Unit` 타입에 속한 값은 단 하나뿐이며 그 이름도 `Unit`이다.
  * `Unit` 타입의 함수는 `Unit` 값을 *묵시적*으로 반환한다.

##### Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다

* 예외를 던져 테스트를 실패시키거나 무한 루프를 도는 함수 등 정상적으로 끝나지 않는 함수를 호출하는 코드를 분석하는 경우 함수가 정상적으로 끝나지 않는다는 사실을 알리기 위해 `Nothing` 이라는 특별한 반환 타입을 사용할 수 있다.
* 아무 값도 포함하지 않으므로 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.
* `Nothing`을 반환하는 함수를 엘비스 연산자의 우항에 사용해서 전제 조건을 검사할 수 있다.
  * `val address = company.address ?: fail("No addres")`



#### 6.3 컬렉션과 배열

##### 널 가능성과 컬렉션

* 타입 인자로 쓰인 타입에도 `?`를 붙여 널을 저장할 수 있음을 나타낼 수 있다.
  * `List<Int?>`는 `Int?` 타입의 값을 저장할 수 있다. 즉, 리스트 안의 각 값이 널이 될 수 있다. 리스트 자체는 널이 아니다.
  * `List<Int>?`는 전체 리스트가 널이 될 수 있다는 의미이다.
* 널이 될 수 있는 값으로 이뤄진 컬렉션으로 널 값을 걸러내는 경우가 자주 있어 코틀린 표준 라이브러리는 그런 일을 하는 `filterNotNull`이라는 함수를 제공한다.
  * 컬렉션 안에 널이 들어 있지 않음을 보장해 주므로 이 함수의 결과값은 `List<Int>` 타입이다.

##### 읽기 전용과 변경 가능한 컬렉션

* 코틀린 컬렉션과 자바 컬렉션을 나누는 가장 중요한 특성 하나는 코틀린에서는 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리했다는 점이다.
  * `Collection` 인터페이스를 사용하면 컬렉션 안의 원소에 대해 이터레이션하고, 컬렉션의 크기를 얻고, 어떤 값이 컬렉션 안에 들어 있는지 검사하고, 컬렉션에서 데이터를 읽는 여러 다른 연산을 수행할 수 있다. 그러나 데이터를 추가하거나 제거할 수는 없다.
  * 컬렉션의 데이터를 수정하려면 `MutableCollection` 인터페이스를 사용한다. `Collection`을 확장하면서 원소를 추가하거나 제거하는 메소드들을 제공한다.
* 읽기 전용 인터페이스를 사용하는 것을 일반적인 규칙으로 하고 변경할 필요가 있을 때만 변경 가능한 버전을 사용하는 것이 좋다.
  * `MutableCollection`을 인자로 받는 함수에는 읽기 전용 컬렉션을 전달하면 컴파일 오류가 발생한다.
* 읽기 전용 컬렉션이라고 해도 꼭 변경 불가능한 컬렉션일 필요는 없고, 변경 가능한 컬렉션 참조가 존재할 수 있다.
  * 따라서 읽기 전용 컬렉션이 항상 thread safe하지는 않다.

##### 코틀린 컬렉션과 자바

* 코틀린의 읽기 전용과 변경 가능 인터페이스의 기본 구조는 `java.util` 패키지에 있는 자바 컬렉션 인터페이스의 구조를 그대로 옮겨 놓았다.
  * 변경 가능한 각 인터페이스는 자신과 대응하는 읽기 전용 인터페이스를 확장한다.
  * 읽기 전용 인터페이스에는 컬렉션을 변경할 수 있는 모든 요소가 빠져 있다.
* `java.util.ArrayList`와 `java.util.HashSet` 클래스를 코틀린에서는 `MutableList`와 `MutableSet` 인터페이스를 상속한 것처럼 취급한다.
  * 코틀린 상위 타입을 갖는 것처럼 취급해 자바 호환성을 제공하고 읽기 전용 인터페이스와 변경 가능 인터페이스를 분리한다.
* 컬렉션과 마찬가지로 `Map` 클래스도 `Map`과 `MutableMap`이라는 버전으로 나타나며, 각 타입으로 컬렉션을 생성하는 함수가 제공된다.
* 자바 메소드를 호출하되 컬렉션을 인자로 넘겨야 한다면 따로 변환하거나 복사하는 등의 추가 작업 없이 직접 컬렉션을 넘기면 된다.
  * 자바는 읽기 전용 컬렉션과 변경 가능 컬렉션을 구분하지 않으므로 코틀린에서 읽기 전용으로 선언된 객체가 자바 코드에서 변경되어도 막을 방법이 없다.
  * 컬렉션을 자바로 넘기는 코틀린 프로그램을 작성한다면 호출하려는 자바 코드가 컬렉션을 변경할지 여부에 따라 올바른 파라미터 타입을 사용할 책임은 프로그래머에게 있다.

##### 컬렉션을 플랫폼 타입으로 다루기

* 자바 코드에서 정의한 타입을 코틀린에서는 플랫폼 타입으로 보며, 플랫폼 타입의 경우 코틀린 쪽에는 널 관련 정보가 없다.
  * 컴파일러는 코틀린 코드가 어느 쪽이든 사용할 수 있게 한다.
* 마찬가지로 자바 쪽에서 선언한 컬렉션 타입 변수를 코틀린에서는 플랫폼 타입으로 본다.
  * 컬렉션 타입이 시그니처에 들어간 자바 메소드 구현을 오버라이드하는 경우 읽기 전용과 변경 가능 컬렉션의 차이가 문제가 된다.
  * 플랫폼 타입에서 널 가능성을 다룰 때처럼 오버라이드하려는 메소드의 자바 컬렉션 타입을 어떤 코틀린 컬렉션 타입으로 표현할지 결정해야 한다.
  * 컬렉션이 널이 될 수 있는지, 컬렉션의 원소가 널이 될 수 있는지, 오버라이드하는 메소드가 컬렉션을 변경할 수 있는지를 고려하여 코틀린에서 사용할 컬렉션 타입에 반영해야 한다.
* 호출하는 쪽에서 항상 오류 메시지를 받아야 하므로 1. `List<String>`은 널이 되면 안 되며,  2. `error`의 언소는 널이 될 수도 있고 구현 코드에서 원소를 추가할 수 있어야 하므로 3. `List<String>`은 변경 가능한 에러 데이터 파서를 코틀린과 구현하면 다음과 같다.

```kotlin
class PersonParser: DataParser<Person> {
  override fun parseData(input: String,
     output: MutableList<Person>,
     errors: MutableList<String?>) {
    // ...
  }
}
```

##### 객체의 배열과 원시 타입의 배열

* 코틀린 배열은 타입 파라미터를 받는 클래스다. 배열의 원소 타입은 타입 파라미터에 의해 정해진다.
  * `arrayOf` 함수에 원소를 넘기는 방식, `arrayOfNulls` 함수에 정수 값을 인자로 넘기는 방식, `Array` 생성자에 배열 크기와 람다를 인자로 받아 람다를 호출하여 배열 원소를 초기화하는 방식 등으로 배열을 만들 수 있다.
* 코틀린에서는 배열을 인자로 받는 자바 함수를 호출하거나 `varang` 파라미터를 받는 코틀린 함수를 호출하기 위해 배열을 만든다.
  * 데이터가 이미 컬렉션에 들어 있다면 컬렉션을 배열로 변환해야 한다. `toTypedArray` 메소드를 사용하면 컬렉션을 배열로 바꿀 수 있다.

```kotlin
val strings = listOf("a", "b", "c")
println("%s/%s/%s".format(*strings.toTypedArray()))
```

* 배열의 타입 인자도 항상 객체 타입이 된다.
  * `Array<Int>` 타입을 선언하면 그 배열은 박싱된 정수의 배열(`java.lang.Integer[]`)이다.
  * 코틀린은 원시 타입의 배열을 표현하는 별도 클래슬르 각 원시 타입마다 하나씩 제공한다. `Int` 타입의 배열은 `IntArray`다. 자바 원시 타입 배열(`int[]`)로 컴파일된다.
* 원시 타입의 배열을 만드는 방법
  * 각 배열 타입의 생성자는 `size` 인자를 받아 해상 원시 타입의 디폴트 값으로 초기화된 `size` 크기의 배열을 반환
  * 팩토리 함수(`intArrayOf` 등)는 여러 값을 가변 인자로 받아 그런 값이 들어간 배열 반환
  * 크기와 람다를 인자로 받는 생성자 사용
  * 박싱된 값이 들어 있는 컬렉션이나 배열이 있다면 `toIntArray` 등의 변환 함수를 사용해 박싱하지 않은 값이 들어 있는 배열로 변환 가능

```kotlin
val fizeZeros = IntArray(5)
val fizeZerosToo = intArrayOf(0, 0, 0, 0, 0)
val squares = IntArray(5) { i -> (i + 1) * (i + 1) }
```

* 코틀린 표준 라이브러리는 배열 기본 연산에 더해 컬렉션에 사용할 수 있는 모든 확장 함수를 배열에도 제공한다.
  * 이런 함수가 반환하는 값은 배열이 아니라 리스트이다.
