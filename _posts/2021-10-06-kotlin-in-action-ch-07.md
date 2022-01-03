---
title: "Kotlin in Action 7장; 연산자 오버로딩과 기타 관례"
categories:
  - Language
tags:
  - kotlin
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 7. 연산자 오버로딩과 기타 관례

* 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해 주는 기법을 코틀린에서는 관례라고 부른다.
* 코틀린은 이러한 관례에 의존한다.

#### 7.1 산술 연산자 오버로딩

##### 이항 산술 연산자 오버로딩

```kotlin
data class Point(val x: Int, val y: Int) {
  operator fun plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
  }
}
```

* 함수 앞에 `operator` 키워드를 붙인다. 연산자를 오버로딩하는 함수 앞에는 꼭 `operator`가 있어야 한다.
* `operator` 변경자를 추가해 함수를 선언하고 나면 기호로 두 객체에 해당하는 연산을 수행할 수 있다.
* 연산자를 멤버 함수로 만드는 대신 확장 함수로 정의할 수도 있다.
  * 외부 함수의 클래스에 대한 연산자를 정의할 때는 관례를 따르는 이름의 확장 함수로 구현하는 게 일반적인 패턴이다.
* 코틀린에서는 기본적인 사칙연산자와 `mod` 연산자를 오버로딩할 수 있다.

> 자바를 코틀린에서 호출하는 경우 함수 이름이 코틀린의 관례에 맞아 떨어지기만 하면 연산자 식을 사용해 함수를 호출할 수 있다. 이름이 다르다면 관례에 맞는 이름을 가진 확장 함수를 작성하고 연산을 기존 자바 메소드에 위임한다. 긴 이름(FQN)을 사용하면 코틀린 연산자를 자바에서 호출할 수 있다.

* 피연산자는 같은 타입일 필요는 없다.
  * 코틀린 연산자가 자동으로 교환 법칙을 지원하지는 않는다.
  * 반대로도 사용할 수 있게 하고 싶다면 대응하는 연산자 함수를 정의해야 한다.
* 연산자 함수의 반환 타입이 꼭 두 피연산자 중 하나와 일치해야만 하는 것도 아니다.
* `operator` 함수도 오버로딩할 수 있으므로 이름은 같지만 파라미터 타입이 서로 다른 연산자 함수를 여럿 만들 수 있다.

> 코틀린은 표준 숫자 타입에 대해 비트 연산자를 정의하지 않는다. 대신에 중위 연산자 표기법을 지원하는 일반 함수를 사용해 비트 연산을 수행한다. (Ex - `0x0F and 0xF0`)

##### 복합 대입 연산자 오버로딩

* 변수가 변경 가능한 경우에만 복합 대입 연산자를 사용할 수 있다.
* 복합 대입 연산이 객체에 대한 참조를 다른 참조로 바꾸기보다 원래 객체의 내부 상태를 변경하게 만들고 싶을 수 있다. 
  * 변경 가능한 컬렉션에 원소를 추가하는 경우가 예시가 될 수 있다.
  * 변환 타입이 `Unit`인 `plusAssign` 함수를 정의하면 코틀린은 `+=` 연산자에 그 함수를 사용한다.
* 이론적으로는 코드에 있는 `+=`를 `plus`와 `plusAssign` 양쪽으로 컴파일할 수 있지만 어떤 클래스가 이 두 함수를 모두 정의하고 둘 다 `+=`에 사용 가능한 경우 컴파일러는 오류를 보고한다.
  * 두 연산을 동시에 정의하지 않는 게 좋다.
* 코틀린 표준 라이브러리는 이항 산술 연산자의 경우 항상 새로운 컬렉션을 반환하고, 복합 대입 연산자의 경우 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화시킨다.
  * 읽기 전용 컬렉션에서 복합 대입 연산자는 변경을 적용한 *복사본*을 반환한다.

##### 단항 연산자 오버로딩

* 이항 연산자와 마찬가지로 미리 정해진 이름의 함수를 `operator`로 표시하면 된다.

```kotlin
operator fun Point.unaryMinus(): Point {
  return Point(-x, -y)
}
```

* 단항 연산자를 오버로딩하기 위해 사용하는 함수는 인자를 취하지 않는다.
* `inc`나 `dec` 함수를 정의해 증가/감소 연산자를 오버로딩하는 경우 컴파일러는 일반적인 값에 대한 전위와 후위 증가/감소 연산자와 같은 의미를 제공한다.



#### 7.2 비교 연산자 오버로딩

##### 동등성 연산자: equals

* `==` 연산자 호출을 코틀린에서는 `equals` 메소드 호출로 컴파일한다.
* `!=` 연산자를 사용하는 식도 `equals` 호출로 컴파일된다.
* 위 두 연산자는 내부에서 인자가 널인지 검사하므로 다른 연산과 달리 널이 될 수 있는 값에도 적용 가능하다.
  * 널이 아닌 경우에만 `equals`를 호출한다. 둘 다 널인 경우는 결과가 `true`가 된다.
* 식별자 비교 연산자(`===`) 를 사용해 `equals`의 파라미터가 수신 객체와 같은지 살펴볼 수 있다.
  * 자바 `==` 연산자와 같다.
  * 서로 같은 객체를 가리키는지(원시 타입인 경우 두 값이 같은지) 비교한다.
  * 이 연산자를 오버로딩할 수는 없다.
* `equals`는 `Any`에 정의된 메소드이므로 `override` 가 붙어 있다.
  * 따라서 따로 `operator`를 붙일 필요 없다.
  * `Any`에서 상속받은 `equals` 가 확장 함수보다 우선순위가 높기 때문에 `equals`를 확장 함수로 정의할 수 없다.

##### 순서 연산자: compareTo

* 자바와 똑같은 `Comparable` 인터페이스를 지원한다. 이 인터페이스 안에 있는 `compareTo` 메소드를 호출하는 관례를 제공한다.
* 비교 연산자는 `compareTo` 호출로 컴파일된다.
* 정의된 `Comparable` 인터페이스를 자바 쪽의 컬렉션 정렬 메소드 등에도 사용할 수 있다.
  * `compareTo`에도 `operator` 변경자가 붙어 있으므로 하위 클래스의 오버라이딩 함수에 `operator` 를 붙일 필요가 없다.
* 코틀린 표준 라이브러리의 `compareValuesBy`는 첫 번째 비교 함수에 두 객체를 넘겨 객체들이 같은지 확인하고 같다면 두 번째 비교 함수를 통해 두 객체를 비교함으로써 `compareTo` 정의를 쉽게 할 수 있게 해 준다.
* `Comparable` 인터페이스를 구현하는 모든 자바 클래스를 코틀린에서는 간결한 연산자 구문으로 비교할 수 있다.



#### 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례

##### 인덱스로 원소에 접근: get과 set

* 코틀린은 인덱스 연산자(`[]`)도 관례를 따른다. 
  * 인덱스 연산자를 사용해 원소를 읽는 연산은 `get` 연산자 메소드로 변환되고, 원소를 쓰는 연산은 `set` 연산자 메소드로 변환된다.
  * `get` 메소드를 만들고 `operator` 를 붙이면 연산자 메소드로 동작한다.
* 인덱스에 해당하는 컬렉션 원소를 쓰고 싶을 때는 `set` 이라는 함수를 정의하면 된다.
  * 각괄호를 사용한 대입문은 `set` 함수 호출로 컴파일된다.

##### in 관례

* `in` 은 객체가 컬렉션에 들어 있는지 검사한다. `in` 연산자와 대응하는 함수는 `contains`다. 
* `in`의 우항에 있는 객체는 `contains` 메소드의 수신 객체가 되고,  `in`의 좌항에 있느 객체는 `contains` 메소드에 인자로 전달된다.
* `a..b`은 닫힌 범위를 의미하고 `a until b`는 열린 범위를 의미한다.

##### rangeTo 관례

* 범위를 만들려면 `..` 구문을 사용해야 한다.
* `..` 연산자는 `rangeTo` 함수를 간략하게 표현하는 방법이다.
  * 범위를 반환한다.
  * `Comparable` 인터페이스를 구현하면 `rangeTo`를 정의할 필요가 없다.
  * 코틀린 표준 라이브러리에는 모든 `Comparable` 객체에 대해 적용 가능한 `rangeTo` 함수가 있다.
* 범위 연산자는 우선순위가 낮아 범위의 메소드를 호출하려면 범위를 괄호로 둘러싸야 한다.

##### for 루프를 위한 iterator 관례

* 코틀린의 `for` 루프는 `in` 연산자를 사용하지만 범위 검사와 의미가 다르다.
* `for (x in list)`  같은 문장은 `list.iterator()` 를 호출해서 이터레이터를 얻은 다음 그 이터레이터에 대해 `hasNext`와 `next()` 호출을 반복하는 식으로 변환된다.
* 이것 역시 관례이므로 `iterator` 메소드를 확장 함수로 정의할 수 있다. 클래스 안에 직접 구현도 가능하다.

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    object : Iterator<LocalDate> {
      var current = start
      
      override fun hasNext() = current <= endInclusive
      
      override fun next() = current.apply { current = plusDays(1) }
    }
}

val newYear = LocalDate.ofYearDay(2022, 1)
val daysOff = newYear.minusDays(1)..newYear
for (dayOff in doaysOff) { /* do something */ }
```



#### 7.4 구조 분해 선언과 component 함수

```kotlin
val p = Point(10, 20)
val (x, y) = p
```

* 구조 분해는 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.
* 내부에서 구조 분해 선언은 관례를 사용한다
  * 각 변수를 초기화하기 위해 `componentN`이라는 함수를 호출한다.
  * `data` 클래스의 주 생성자에 들어 있는 프로퍼티에 대해서는 컴파일러가 자동으로 `componentN` 함수를 만들어 준다.
  * 데이터 타입이 아닌 경우 함수를 구현할 수 있다.

```kotlin
class Point(val x: Int, val y: Int) {
  operator fun component1() = x
  operator fun component2() = y
}
```

##### 구조 분해 선언과 루프

* 변수 선언이 들어갈 수 있는 곳이라면 어디든 구조 분해 선언을 사용할 수 있다.

```kotlin
for((key, value) in map) {
  println("$key -> $value")
}
```



#### 7.5 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

* 위임은 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말한다.
* 이때 작업을 처리하는 도우미 객체를 위임 객체라고 부른다. 이 패턴을 프로퍼티에 적용해 접근자 기능을 도우미 객체가 수행하도록 위임한다.

##### 위임 프로퍼티 소개

```kotlin
class Foo {
  var p: Type by Delegate()
}
```

* `p` 프로퍼티는 접근자 로직을 다른 객체에게 위임한다.
  * `by` 뒤에 있는 식을 계산해서 위임에 쓰일 객체를 얻는다.
  * 프로퍼티 위임 객체가 따라야 하는 관례를 따르는 모든 객체를 위임에 사용할 수 있다.
  * 프로퍼티 위임 관례를 따르는 위임 객체 클래스는 `getValue`와 `setValue` 메소드를 제공해야 한다.
  * 프로퍼티 위임 객체를 사용하는 프로퍼티는 일반 프로퍼티처럼 쓸 수 있지만 실제 게터와 세터는 위임 프로퍼티 객체에 있는 메소드를 호출한다.

##### 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 지연

* 지연 초기화는 객체의 일부분을 초기화하지 않고 남겨 뒀다가 실제로 그 부분의 값이 필요할 경우 초기화할 때 쓰이는 패턴이다.
* 뒷받침하는 프로퍼티 기법을 사용해서 값을 저장하는 프로퍼티와 그 프로퍼티에 대한 읽기 연산을 제공하는 다른 프로퍼티를 두는 방법을 사용할 수 있지만 이는 코드가 복잡해지고 스레드 안전하지 않다는 단점이 있다.
* 위임 프로퍼티는 이를 훨씬 더 간단하게 구현하게 해 준다.
  * 데이터 저장 시 쓰이는 뒷받침하는 프로퍼티와 값이 한 번만 초기화됨을 보장하는 게터 로직을 함께 캡슐화해 준다.
* `lazy` 함수는 코틀린 관례에 맞는 시그니처의 `getValue` 메소드가 들어 있는 객체를 반환한다.
  * 따라서 `lazy`를 `by` 키워드와 함께 사용해 위임 프로퍼티를 만들 수 있다.
  * `lazy` 함수의 인자는 값을 초기화할 때 호출할 람다다.
  * 기본적으로 스레드 안전하다.

```kotlin
class Person(val name: String) {
  val emails by lazy { loadEmails(this) }
}
```

##### 위임 프로퍼티 구현

* 코틀린 관례에 사용하는 다른 함수와 마찬가지로 `getValue`와 `setValue`에도 `operator` 변경자가 붙는다.
* `getValue`와 `setValue`는 프로퍼티가 포함된 객체와 프로퍼티를 표현하는 객체를 파라미터로 받는다.
  * 코틀린은 `KProperty` 타입의 객체를 사용해 프로퍼티를 표현한다.
  * `KProperty.name`을 통해 메소드가 처리할 프로퍼티 이름을 알 수 있다.

````kotlin
class ObservableProperty(
	 var propValue: Int, val changeSupport: PropertyChangeSupport
) {
  operator fun getValue(p: Person, prop: KProperty<*>): Int = propValue
  operator fun setValue(p: Person, prop: KProperty<*>, newValue: Int) {
    val oldValue = propValue
    propValue = newValue
    changeSupport.firePropertyChange(prop.name, oldValue, newValue)
  }
}
````

```kotlin
class Person(
	 val name: String, age: Int, salary: Int
) : PropertyChangeAware() {
   var age: Int by ObservableProperty(age, changeSupport)
   var salary: Int by ObservableProperty(salary, changeSupport)
}
```

* `by` 키워드를 사용해 위임 객체를 지정하면 코틀린 컴파일러가 자동으로 처리해 준다.
  * `by` 오른쪽에 오는 객체를 위임 객체라고 부르며, 코틀린은 위임 객체를 감춰진 프로퍼티에 저장하고 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 `getValue`와 `setValue`를 호출해 준다.
  * `by` 오른쪽 식에는 함수 호출, 다른 프로퍼티, 다른 식 등이 올 수 있다. 다만 우항에 있는 식을 계산한 결과인 객체는 컴파일러가 호출할 수 있는 올바른 타입의 `getValue`와 `setValue`를 제공해야 한다.

##### 위임 프로퍼티 컴파일 규칙

* 컴파일러는 위임 프로퍼티에 해당하는 클래스의 인스턴스를 감춰진 프로퍼티에 저장하며 그 감춰진 프로퍼티를 `<delegate>` 라는 이름으로 부른다.
* 프로퍼티를 표현하기 위해 `KProperty` 타입의 객체를 사용한다. 이 객체를 `<property>` 라고 부른다.
* 컴파일러는 `class C { val prop: Type by Mydelegate() }` 라는 코드에 대해 아래 코드를 생성한다.

````kotlin
class C {
  private val <delegate> = Mydelegate()
  var prop: Type
  get() = <delegate>.getValue(this, <property>)
  set(value: Type) = <delegate>.setValue(this, <property>, value)
}
````

##### 프로퍼티 값을 맵에 저장

* 자신의 프로퍼티를 동적으로 정의할 수 있는 객체를 만들 때 위임 프로퍼티를 활용하는 경우가 있다. 그런 객체를 확장 가능한 객체라고 부르기도 한다.
* 정보를 모두 맵에 저장하되 그 맵을 통해 처리하는 프로퍼티를 통해 필수 정보를 제공하는 방법을 사용할 수 있다.

```kotlin
class Person {
  // 추가 정보
  private val _attributes = hashMapOf<String, String>()
  fun setAttribute(attrName: String, value: String) { // 추가 데이터를 저장하는 일반적인 API
    _attributes[attrName] = value
  }
  
  // 필수 정보
  val name: String
  get() = _attributes["name"]!! // 특정 프로퍼티를 처리하는 구체적인 개별 API
}
```

* 위임 프로퍼티를 활용하게 변경할 수 있다.

```kotlin
class Person {
  private val _attributes = hashMapOf<String, String>()
  fun setAttribute(attrName: String, value: String) {
    _attributes[attrName] = value
  }
  
  val name: String by _attributes // 표준 라이브러리가 Map, MutableMap에 대해 getValue, setValue 제공하기 때문에 가능
}
```

##### 프레임워크에서 위임 프로퍼티 활용

```kotlin
object Users : IdTable() { // 데이터베이스 테이블에 해당
  val name = varchar("name", length = 50).index() // 테이블 컬럼에 해당
  val age = integer("age")
}

class User(id: EntityID) : Entity(id) { // 테이블에 들어 있는 구체적인 엔티티에 해당
  var name: String by Users.name // 데이터베이스 name 컬럼에서 가져온다
  var age: Int by Users.age
}
```

* 위 예에서 각 엔티티 속성은 위임 프로퍼티며 컬럼 객체(`Users.name`, `Users.age`)를 위임 객체로 사용한다.
* 컬럼 클래스 안에 `getValue` 와 `setValue` 메소드를 정의하면 컬럼 프로퍼티를 위임 프로퍼티에 대한 위임 객체로 사용할 수 있고, 값을 가져오거나 세팅할 때 이 메소드들을 사용하게 된다.
