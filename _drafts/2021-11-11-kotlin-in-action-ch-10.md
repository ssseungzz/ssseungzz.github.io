---
title: "Kotlin in Action 10장; 애노테이션과 리플렉션"
categories:
  - Language
tags:
  - kotlin
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 10. 애노테이션과 리플렉션

#### 10.1 애노테이션 선언과 적용

##### 애노테이션 적용

* 자바와 같은 방법으로 애노테이션을 사용할 수 있다.
  * `replaceWith` 파라미터를 통해 옛 버전을 대신할 수 있는 패턴을 제시할 수 있다.
* 애노테이션에 인자를 넘길 때는 일반 함수와 마찬가지로 괄호 안에 인자를 넣는다.
  * 클래스를 애노테이션 인자로 지정할 때는 `@MyAnnotation(MyClass:class)` 처럼 `::class` 를 클래스 이름 뒤에 넣는다.
  * 다른 애노테이션을 인자로 지정할 떄는 인자로 들어가는 애노테이션의 이름 앞에 `@`를 넣지 않아야 한다.
  * 배열을 인자로 지정하려면 `@RequestMapping(path = arrayOf("/foo", "/bar"))` 처럼 `arrayOf` 함수를 사용한다.
* 애노테이션 인자를 컴파일 시점에 알 수 있어야 하므로 임의의 프로퍼티를 인자로 지정할 수는 없다. 프로퍼티를 애노테이션 인자로 사용하려면 그 앞에 `const` 변경자를 붙여야 한다.

##### 애노테이션 대상

* 사용 지점 대상 선언으로 애노테이션을 붙일 요소를 정할 수 있다.
  * 사용 지점 대상은 `@` 기호와 애노테이션 이름 사이에 붙으며 애노테이션 이름과는 콜론으로 분리된다.
  * `@get:Rule` 에서 `get` 은 `@Rule` 애노테이션을 프로퍼티 게터에 적용하라는 뜻이다.
* 코틀린으로 애노테이션을 선언하면 프로퍼티에 직접 적용할 수 있는 애노테이션을 만들 수 있다.
* 사용 지점 대상을 지정할 때 지원하는 대상 목록은 아래와 같다.
  * `property`: 프로퍼티 전체, 자바에서 선언된 애노테이션에는 이 사용 지점 대상을 사용할 수 없다.
  * `field`: 프로퍼티에 의해 생성되는 (뒷받침하는) 필드
  * `get`: 프로퍼티 게터
  * `set`: 프로터피 세터
  * `receiver`: 확장 함수나 프로퍼티의 수신 객체 파라미터
  * `param`: 생성자 파라미터
  * `setparam`: 세터 파라미터
  * `delegate`: 위임 프로퍼티의 위임 인스턴스를 담아 둔 필드
  * `file`: 파일 안에 선언된 최상위 함수와 프로퍼티를 담아 두는 클래스

##### 애노테이션을 사용한 JSON 직렬화 제어

* 애노테이션을 활용해 객체를 직렬화하거나 역직렬화하는 방법을 제어할 수 있다.
  * 애노테이션을 사용하면 프로퍼티를 직렬화하며 프로퍼티 이름을 키로 사용한느 등의 동작을 변경할 수 있다.
* `@JsonExclude`: 직렬화나 역직렬화 시 프로퍼티 무시
* `@JsonName`: 프로퍼티를 표현하는 키/값 쌍의 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름 사용

```kotlin
data class Person(
	@JsonName("alias") val firstName: String,
  @JsonExclude val age: Int? = null
)
```

##### 애노테이션 선언

```kotlin
annotation clas JsonExclude
```

* 애노테이션 클래스는 오직 선언이나 식과 관련 있는 메타데이터의 구조를 정의하기 때문에 내부에 아무 코드도 들어 있을 수 없다.
* 파라미터가 있는 애노테이션을 정의하려면 애노테이션 클래스의 주 생성자에 파라미터를 선ㄷ언한다.
  * 일반 클래스의 주 생성자 선언 구문을 똑같이 사용하나 모든 파라미터 앞에 `val` 을 붙여야만 한다.

##### 메타애노테이션: 애노테이션을 처리하는 방법 제어

* 애노테이션에 적용할 수 있는 애노테이션을 메타애노테이션이라고 부른다.

* `@Target` 메타애노테이션은 애노테이션을 적용할 수 있는 요소의 유형을 지정한다.

  * 지정하지 않으면 모든 선언에 적용할 수 있는 애노테이션이 된다.
  * 메타애노테이션을 직접 만들어야 한다면 `@Target(AnnotationTarget.ANNOTATION_CLASS)` 로 설정한다.
  * 대상을 `PROPERTY` 로 지정한 애노테이션을 자바 코드에서 사용할 수 없다. 사용하려면. `AnnotationTarget.FIELD` 를 두 번째 대상으로 추가해야 한다.

  >  `@Retention` 은 정의 중인 애노테이션 클래스를 소스 수준에서 유지할지, `.class` 파일에 저장할지, 실행 시점에 리플렉션을 사용해 접근할 수 있게 할지를 지정하는 메타애노테이션이다. 코틀린에서는 기본적으로 `RUNTIME`으로 지정한다.

##### 애노테이션 파라미터로 클래스 사용

* 클래스 참조를 인자로 받는 애노테이션을 정의할 수 있다.

```kotlin
annotation class DeserializeInterface(val targetClass: KClass<out Any>)
```

* `KClass` 는 자바 `java.lang.Class` 타입과 같은 역할을 하는 코틀린 타입이다. 코틀린 클래스에 대한 참조를 저장할 때 사용한다.
* `KClass` 의 타입 파라미터는 이 `KClass` 인스턴스가 가리키는 코틀린 타입을 지정한다.
  * 예를 들어 `Company::class` 의 타입은 `KClass<Company>` 이며 이 타입은 `KClass<out Any>` 의 하위 타입이다.
  * `out` 키워드를 사용하면 모든 코틀린 타입 `T`에 대해 `KClass<T>` 가 `KClass<out Any>` 의 하위 타입이 된다.

##### 애노테이션 파라미터로 제네릭 클래스 받기

* 애노테이션 파라미터로 제네릭 클래스를 받을 때, 제네릭 클래스는 타입 파라미터가 있으므로 제네릭 클래스 타입을 참조하려면 항상 타입 인자를 제공해야 한다. 
  * 하지만 애노테이션이 어떤 타입에 대해 쓰일지 알 수 없으므로 스타 프로젝션을 사용할 수 있다.

````kotlin
interface ValueSerializer<T> {
  fun toJsonValue(value: T): Any?
  fun fromJsonValue(jsonValue: Any?): T
}

annotation class CustomSerializer(
	val serializerClass: KClass<out ValueSerializer<*>>
)

data class Person {
  val name: String,
  @CustomSerializer(DateSerializer::class) val birthDate: Date
}
````

* 클래스를 인자로 받아야 하면 애노테이션 파라미터 타입에 `KClass<out 허용할 클래스 이름>` 을 쓴다.
* 제네릭 클래스를 인자로 받아야 하면 `KClass<out 허용할 클래스 이름<*>>` 처럼 허용할 클래스의 이름 뒤에 스타 프로젝션을 덧붙인다.



#### 10.2 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

* 리플렉션은 실행 시점에 동적으로 객체의 프로퍼티와 메소드에 접근할 수 있게 해 주는 방법이다.
* 코틀린에서 리플렉션을 사용하려면 두 가지 서로 다른 리플렉션 API를 다뤄야 한다.
  * 첫 번째는 `java.lang.reflect` 패키지를 통해 제공되는 표준 리플렉션이다. 코틀린 클래스는 일반 자바 바이트코드로 컴파일되므로 자바 리플렉션 API도 코틀린 클래스를 컴파일한 바이트코드를 완벽하게 지원한다.
  * 두 번째는 코틀린이 `kotlin.reflect` 패키지를 통해 제공하는 API로 코틀린 고유 개념에 대한 리플렉션을 제공한다.

##### 코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty

* `KClass` 를 사용하면 클래스 안에 있는 모든 선언을 열거하고 각 선언에 접근하거나 클래스의 상위 클래스를 얻을 수 있다.
  * `MyClass::class` 라는 식을 쓰면 `KClass` 의 인스턴스를 얻을 수 있다.
* 실행 시점에 객체의 클래스를 얻으려면 객체의 `javaClass` 프로퍼티를 이용해 객체의 자바 클래스를 얻고 `.kotlin` 확장 프로퍼티를 통해 `KClass` 를 얻어 자바에서 코틀린 리플렉션 API로 옮겨 올 수 있다.
* `KClass`의 모든 멤버의 목록은 `KCallable` 인스턴스의 컬렉션이다.
  * `KCallable` 은 함수와 프로퍼티를 아우르는 공통 상위 인터페이스이다. 안에는 `call` 메소드가 들어 있는데, 함수나 프로퍼티의 게터를 호출할 수 있다.
  * `call` 을 사용할 때는 함수 인자를 `varang` 리스트로 전달한다.

```kotlin
fun foo(x: Int) = println(x)
val kFunction = ::foo
KFunction.call(42)
```

* 구체적인 메소드를 사용해서 함수를 호출할 수도 있다.
  * `KFunction1<Int, Unit>` 과 같이 함수의 파라미터가 1개, 반환 타입이 `Unit` 이라는 정보를 알 수 있는 인터페이스를 사용한다.
  * 위의 형태의 인터페이스를 통해 함수를 호출하려면 `invoke` 메소드를 사용해야 한다. 이 메소드는 정해진 개수의 인자만을 받아들인다. 인자 타입은 인터페이스의 첫 번째 타입 파라미터와 같다.
  * 인자 타입과 반환 타입을 모두 안다면 `invoke` 메소드를 호출하는 게 낫다. `call` 메소드는 모든 타입의 함수에 적용할 수 있는 일반적인 메소드지만 타입 안정성을 보장하지는 않는다.
* * `KProperty` 의 `call` 메소드를 호출할 수 있지만 프로퍼티 인터페이스는 프로퍼티 값을 얻는 더 좋은 방법으로 `get` 메소드를 제공한다.
    * 이 메소드에 접근하려면 프로퍼티가 선언된 방법에 따라 올바른 인터페이스를 사용해야 한다.
    * 최상위 프로퍼티는 `KProperty0` 인터페이스의 인스턴스로 표현되며 인자가 없는 `get` 메소드가 있다.
    * 멤버 프로퍼티는 `KProperty1` 인스턴스로 표현되며 인자가 1개인 `get` 메소드가 있다.  - 멤버 프로퍼티는 어떤 객체에 속한 프로퍼티이므로 값을 가져오려면 `get` 메소드에게 프로퍼티를 얻고자 하는 객체 인스턴스를 넘겨야 한다.
      * `KProperty1` 은 제네릭 클래스로 첫 번째 타입 파라미터는 수신 객체 타입, 두 번째 타입 파라미터는 프로퍼티 타입을 표현한다.
    * 최상위 수준이나 클래스 안에 정의된 프로퍼티만 리플렉션으로 접근할 수 있다.

##### 리플렉션을 사용한 객체 직렬화 구현

```kotlin
private fun StringBuilder.serializeObject(obj: Any) {
  val kClass = obj.javaClass.kotlin // 객체의 KClass를 얻는다
  val properties = KClass.memberProperties // 클래스의 모든 프로퍼티를 얻는다
  
  properties.joinToStringBuilder(
  		 this, prefix = "{", postfix = "}") { prop -> 
     serializeString(prop.name) // 프로퍼티 이름을 얻는다
     append(": ")
     serializePropertyValue(prop.get(obj)) // 프로퍼티 값을 얻는다
  }
}
```

* 각 프로퍼티가 어떤 타입인지 알 수 없으므로 `prop` 변수의 타입은 `KProperty1<Any, *>` 이며 `get` 메소드는 `Any` 타입의 값을 반환한다.
* 이 경우 수신 객체 타입을 컴파일 시점에 검사할 방법이 없지만 위 코드에서는 어떤 프로퍼티의 `get` 에 넘기는 객체가 바로 그 프로퍼티를 얻어온 객체이므로 항상 프로퍼티 값이 제대로 반환된다.

##### 애노테이션을 활용한 직렬화 제어

* `KAnnotatedElement` 인터페이스에는 `annotations` 프로퍼티가 있다.
  * `annotations` 는 소스코드상에서 해당 요소에 적용된 모든 애노테이션 인스턴스의 컬렉션이다.
  * `findAnnotations`라는 함수를 통해 인자로 전달받은 타입에 해당하는 애노테이션을 반환하게 할 수 있다.


```kotlin
val properties = kClass.memberProperties
      .filter { it.findAnnotation<JsonExclude>() == null }
```

