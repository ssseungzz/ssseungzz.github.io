---
title: "Kotlin in Action 3장; 함수 정의와 호출"
categories:
  - Language
tags:
  - kotlin
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 3. 함수 정의와 호출

* 컬렉션, 문자열, 정규식을 다루기 위한 함수
* 이름 붙인 인자, 디폴트 파라미터 값, 중위 호출 문법 사용
* 확장 함수와 확장 프로퍼티를 사용해 자바 라이브러리 적용
* 최상위 및 로컬 함수와 프로퍼티를 사용해 코드 구조화

#### 3.1 코틀린에서 컬렉션 만들기

```kotlin
val set = hashSetOf(1, 7, 53)
val list = arrayListOf(1, 7, 53)
val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

* `to` 는 언어가 제공하는 키워드가 아니라 함수이다.
* `set.javaClass` 와 같은 메서드를 통해 객체가 속하는 클래스를 알 수 있다. 코틀린은 자신만의 컬렉션을 가지지 않고 표준 자바 컬렉션을 활용한다. 
  * 코틀린 컬렉션은 자바 컬렉션과 똑같지만 더 많은 기능을 쓸 수 있다. (마지막 원소 가져오기, 최댓값 찾기 등)

#### 3.2 함수를 호출하기 쉽게 만들기

* 원하는 형식으로 리스트를 프린트하는 함수를 작성해 본다.

```kotlin
fun <T> joinToString (
  collection: Collection<T>,
  separator: String,
  prefix: String,
  postfix: String
) : String {
  
  val result = StringBuilder(prefix)
  
  for((index, element) in collection.withIndex()) {
    if(index > 0) result.append(separator)
    result.append(element)
  }
  
  result.append(postfix)
  return result.toString()
}
```

* 어떻게 이 함수를 호출하는 문장을 좀 더 깔끔하게 만들 수 있을까?

##### 이름 붙인 인자

* 위의 함수를 `joinToString(collection, "", "", "")` 처럼 호출하면 인자가 많고 역할을 구분하기 어렵다. 코틀린에서는 아래와 같이 조금 더 보기 쉽게 표현할 수 있다.
  * 함수에 전달하는 인자 중 일부 혹은 전부의 이름을 명시할 수 있다.

```kotlin
joinToString(collection, separator = " ", prefix = " ", postfix = ".")
```

> IntelliJ에서는 호출 대상 함수가 파라미터 이름을 추적 가능하다. 다만 함수의 파라미터 이름을 바꿀 때 직접 에디터에서 입력하지 않고 refactor의 기능을 사용해서 바꿔야 한다.
>
> 자바로 작성한 코드를 호출할 때는 이름 붙인 인자를 사용할 수 없다.

##### 디폴트 파라미터 값

* 자바에서는 일부 클래스에서 오버로딩한 메소드가 너무 많아진다는 문제가 있다. 
* 코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있으므로 이런 오버로드 중 상당수를 피할 수 있다.
  * 함수 호출 시 모든 인자를 쓸 수도 있고 일부를 생략할 수도 있다.

```kotlin
fun <T> joinToString (
  collection: Collection<T>,
  separator: String = ", ", // 디폴트 값 지정
  prefix: String = "",
  postfix: String = ""
) : String
```

* 일반 호출 문법을 사용하려면 함수 선언과 같은 순서로 인자를 지정해야 한다. 이름 붙인 인자를 사용하는 경우 인자 목록의 중간에 있는 인자를 생략하고 지정하고 싶은 인자를 이름을 붙여서 순서와 관계없이 지정할 수 있다.
* 함수의 디폴트 파라미터 값은 함수 선언 쪽에서 지정된다. 따라서 어떤 클래스 안에 정의된 함수의 디폴트 값을 바꾸고 그 클래스가 포함된 파일을 재컴파일하면 그 함수를 호출하는 코드 중에 값을 지정하지 않은 모든 인자는 자동으로 바뀐 디폴트 값을 적용받는다.

> 자바에는 디폴트 파라미터 값이라는 개념이 없어 코틀린 함수를 자바에서 호출하는 경우에는 그 코틀린 함수가 디폴트 파라미터 값을 제공하더라도 모든 인자를 명시해야 한다. 자바에서 코틀린 함수를 자주 호출하는 경우 좀 더 편하게 코틀린 함수를 호출할 수 있다. `@JvmOverloads` 어노테이션을 함수에 추가하면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해 준다.

##### 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

* 자바에서는 모든 코드를 클래스의 메소드로 작성해야 하므로 다양한 정적 메소드를 모아 두는 클래스가 생겨났다.
* 코틀린에서는 이런 무의미한 클래스가 필요 없다. 대신 함수를 직접 소스 파일의 최상의 수준에 위치시키면 된다. 다른 패키지에서 함수를 사용하고 싶은 경우 그 함수가 정의된 패키지를 임포트해야 하지만 클래스 이름이 추가로 들어갈 필요는 없다.

```kotlin
package strings

fun joinToString(...): String {...}
```

* JVM이 클래스 안에 들어 있는 코드만을 실행할 수 있기 때문에 컴파일러는 이 파일을 컴파일할 때 새로운 클래스를 정의해 준다. 위의 함수를 컴파일한 결과와 같은 클래스를 자바 코드로 작성하면 아래와 같다.

```java
package strings;

public class JoinKt {
  public static String joinToString(...) { ... }
}
```

* 코틀린 컴파일러가 생성하는 클래스의 이름은 최상위 함수가 들어 있던 코틀린 소스 파일의 이름과 대응한다.
* 코틀린 파일의 최상위 함수는 이 클래스의 정적 메소드가 된다. 따라서 자바에서는 아래와 같이 호출할 수 있다.

````java
import string.JoinKt;

JoinKt.joinToString(list, ", ", "", "")
````

> 코틀란 최상위 함수가 포함되는 클래스의 이름을 바꾸고 싶다면 파일에 `@JvmName` 어노테이션을 추가한다. 어노테이션은 파일의 맨 앞, 패키지 이름 선언 이전에 위치해야 한다.

###### 최상위 프로퍼티

* 프로퍼티도 파일의 최상위 수준에 놓을 수 있다.

```kotlin
var opCount = 0

fun performOperation() {
  opCount++;
}
```

* 이런 프로퍼티의 값은 정적 필드에 저장된다. 최상위 프로퍼티를 활용해 코드에 상수를 추가할 수도 있다.
* 기본적으로 최상위 프로퍼티도 접근자 메소드를 통해 자바 코드에 노출된다. (`val` 의 경우 게터, `var` 의 경우 게터와 세터가 생성)
  * 게터 없이 자연스럽게 사용하려면 상수를 `public static final` 필드로 컴파일해야 하는데, `const` 변경자를 추가하면 가능하다.

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```

* 이는 아래의 자바 코드와 같다.

```java
public static final String UNIX_LINE_SEPARATOR = "\n";
```



#### 3.3 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

* 기존 자바 API를 재작성하지 않고도 코틀린이 제공하는 여러 편리한 기능을 사용하기 위해 확장 함수를 사용할 수 있다.
* 확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.

```kotlin
package strings

fun String.lastChar() Char = this.get(this.length - 1) // String은 수신 객체 타입이고, this는 수신 객체가 된다.
```

* 확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이기만 하면 된다. 클래스 이름을 수신 객체 타입이라 부르며 확장 함수가 호출되는 대상이 되는 값(객체)을 수신 객체라 부른다.
  * 수신 객체 타입은 확장이 정의될 클래스의 타입이며, 수신 객체는 그 클래스에 속한 인스턴스 객체다.

```kotlin
"kotlin".lastChar() 
```

* 직접 작성한 코드가 아니더라도 원하는 메소드를 클래스에 추가할 수 있다. JVM 언어이기만 하면 어떤 언어로 작성되었는지 관계없으며, 자바 클래스로 컴파일한 클래스 파일이 있는 한 그 클래스에 원하는 대로 확장을 추가할 수 있다.
* 확장 함수 본문에 `this`를 쓰거나 생략할 수 있다. 
* 확장 함수 내부에서는 일반적인 인스턴스 메소드의 내부와 마찬가지로 수신 객체의 메소드가 프로퍼티를 바로 사용할 수 있다.
  * 그러나 클래스 내부에서만 사용할 수 있는 `private` 멤버나  `protected` 멤버는 사용할 수 없다.

##### 임포트와 확장 함수

* 확장 함수를 사용하기 위해서는 그 함수를 다른 클래스나 함수와 마찬가지로 임포트해야 한다.

```kotlin
import strings.lastChar

val c = "kotlin".lastChar()
```

* `as` 키워드를 사용하면 임포트한 클래스나 함수를 다른 이름으로 부를 수 있다.
* 한 파일 안에서 여러 패키지에 속해 있는 이름이 같은 함수를 가져와 사용해야 하는 경우 이름을 바꿔서 임포트하면 이름 충돌을 막을 수 있다.

##### 자바에서 확장 함수 호출

* 내부적으로 확장 함수는 수신 객체를 첫 번째 인자로 받는 정적 메소드다. 자바에서 확장 함수를 사용하기 위해서는 단지 정적 메소드를 호출하면서 첫 번째 인자로 수신 객체를 넘기기만 하면 된다.
* 확장 함수가 들어 있는 자바 클래스 이름은 확장 함수가 들어 있는 파일 이름에 따라 결정된다.

```kotlin
char c = StringUtilKt.lastChar("java");
```

##### 확장 함수로 유틸리티 함수 정의

```java
fun <T> Collection<T>.joinToString (
  collection: Collection<T>,
  separator: String = ", ", 
  prefix: String = "",
  postfix: String = ""
) : String {
  
  val result = StringBuilder(prefix)
  
  for((index, element) in collection.withIndex()) {
    if(index > 0) result.append(separator)
    result.append(element)
  }
  
  result.append(postfix)
  return result.toString()
}
```

* `joinToString`을 `Collection`에 있는 클래스의 멤버인 것처럼 호출할 수 있다.
* 확장 함수는 정적 메소드 호출에 대한 문법적인 편의이므로 클래스가 아닌 더 구체적인 타입을 수신 객체 타입으로 지정할 수도 있다.

```kotlin
fun Collection<String>.join(
  separator: String = ", ", 
  prefix: String = "",
  postfix: String = ""
) = joinToString(separator, prefix, postfix)
```

* 확장 함수가 정적 메소드와 같은 특징을 가지므로 확장 함수를 하위 클래스에서 오버라이드할 수 없다.

##### 확장 함수는 오버라이드할 수 없다

```kotlin
open class View {
  open fun click() = println("view clicked")
}

class Button: View() {
  override fun click() = println("button clicked")
}
```

* `Button`이 `View`의 하위 타입이므로 `View` 타입 변수를 선언해도 `Button` 타입 변수를 그 변수에 대입할 수 있지만 특정 메소드를 오버라이드했다면 그 메소드 호출 시 오버라이드한 메소드가 호출된다.
* 그러나 확장은 위와 같이 작동하지 않는다.
* 확장 함수는 클래스의 일부가 아니라 클래스 밖에 선언된다. 이름과 파라미터가 완전히 같은 확장 함수를 가진 기반 클래스와 하위 클래스에 대해 정의해도 실제로는 확장 함수를 호출할 때 수신 객체로 지정한 변수의 정적 타입에 의해 어떤 함수가 호출될지 결정된다.

```kotlin
fun View.showOff() = println("i'm a view")
fun Button.showOff() = println("i'm a button")

val view: View = Button()
view.showOff()
```

* 확장 함수는 첫 번째 인자가 수신 객체인 정적 자바 메소드로 컴파일된다는 사실을 기억하자.

```java
View view = new Button();
ExtensionsKt.showOff(view);
```

> 어떤 클래스를 확장한 함수와 그 클래스의 멤버 함수의 이름과 시그니처가 같다면 멤버 함수가 호출된다. 즉, 멤버 함수의 우선순위가 더 높다.

##### 확장 프로퍼티

* 확장 프로퍼티를 사용하면 기존 클래스 객체에 대한 프로퍼티 형식의 구문으로 사용할 수 있는 API를 추가할 수 있다. 하지만 상태를 저장할 방법이 없기 때문에 실제로 확장 프로퍼티는 아무 상태도 가질 수 없다.
  * 그럼에도 프로퍼티 문법으로 더 짧게 코드를 작성할 수 있어서 편리할 수 있다.

```kotlin
val String.lastChar = Char
	 get() = get(length - 1)
```

* 확장 프로퍼티도 일반적인 프로퍼티와 같은데 수신 객체 클래스가 추가됐다.
* 뒷받침하는 필드가 없어 게터는 꼭 정의를 해야 한다. 초기화 코드에서 계산한 값을 담을 장소도 없으므로 초기화 코드도 쓸 수 없다.

```kotlin
var StringBuilder.lastChar: Char
   get() = get(length - 1)
   set(value: Char) {
     this.setCharAt(length - 1, value)
   }
```

* 자바에서 확장 프로퍼티를 사용하고 싶다면 `StringUtilKt.getLastChar("java")` 처럼 게터나 세터를 명시적으로 호출해야 한다.



#### 3.4 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

