---
title: "Java reflection 알아보기"
categories:
  - Language
tags:
  - java
  - reflection
toc: true
toc_label: "Contents"
toc_sticky: true
---

### Reflection(반영)이란?

* 컴퓨터 프로그램에서 런타임 시점에 사용되는 자신의 구조와 행위를 관리하고 수정할 수 있는 프로세스를 의미한다. "Type introspection"은 객체 지향 프로그램 언어에서 런타임에 객체의 형(type)을 결정할 수 있는 능력을 의미한다. 
  * 런타임에 프로그램의 수행을 수정하고, 관찰하기 위해 사용할 수 있다. 반영 지향적인 프로그램 구성 요소는 내부 코드의 실행을 감시할 수 있고, 구성 요소 자신의 궁극적인 목표에 맞도록 내부를 수정할 수 있다. 런타임에 프로그램 코드를 동적으로 할당하여 이루어진다.
* 대충 런타임에 실행을 뜯어 볼 수 있다는 이야기인 것 같은데, 정확히는 모르겠다.

#### Java의 Reflection

* 자바의 모든 클래스 혹은 인터페이스는 컴파일 과정을 거치면 `.class` 파일이 생성된다. 이 타입은 클래스 혹은 인터페이스 자체의 구성 정보를 담은 오브젝트이다. 
  * `class` 타입의 오브젝트는 클래스의 이름이 무엇인지, 어떤 클래스를 상속했는지, 어떤 인터페이스를 구현했으며 어떤 필드를 가지고 있는지, 각각의 타입은 무엇인지, 메소드는 무엇을 정의했고 메소드의 파라미터와 리턴 타입은 무엇인지 알아낼 수 있다. 말 그대로 모든 구성 정보를 가지고 있는 것이다. 
  * 여기에 필드의 값을 읽고 수정할 수도 있고, 원하는 파라미터 값을 이용해 메소드를 호출할 수 있다.

> 위의 reflection의 정의에서 "구성 요소 자신의 궁극적인 목표에 맞도록 내부를 수정"할 수 있다는 대목이 떠오른다.

* *reflection programming*은 `class` 타입의 오브젝트에 대한 정보-즉, 어떤 클래스에 대한 모든 정보-를 읽어 와 사용자가 그 클래스에 대한 정보가 없더라도 접근할 수 있게 한다.
* 그렇다면 어떻게 `class` 타입의 오브젝트를 생성할 수 있을까?

```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        Class strClass = Class.forName("java.lang.String");
        Constructor[] cons = strClass.getConstructors();
        for (Constructor val : cons) {
            System.out.println(val);
        }
    }
}
```

* 위 코드를 실행하면 `java.lang.String` 클래스 안에 있는 모든 생성자를 보여 준다. 클래스의 이름으로 해당 클래스에 대한 `class` 타입의 오브젝트를 얻을 수 있는 것이다. 이 오브젝트 안에는 클래스에 대한 생성자 정보가 담겨 있으므로, 생성자 정보를 출력할 수 있다.
  * 생성자 정보 외에도 필드 정보, 메소드 정보도 같은 방식으로 가져올 수 있다.

```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        Class strClass = Class.forName("java.lang.String");
        Method lengthMethod = strClass.getMethod("length");
        lengthMethod.invoke("java reflection");
    }
}
```

* `class` 타입의 오브젝트를 통해 해당 클래스의 인스턴스를 생성할 수도 있다.
  * `newInstance` 메소드는 가져온 클래스 정보를 바탕으로 디폴트 생성자를 통해 인스턴스를 만들어 준다.
  * `Object` 형이기 때문에 다운캐스팅이 필요하다.

```java
public class classInstanceTest {
  
  @Test
  void shapeTest() {
    Class shapeClass = Class.forName("test.Shape");
    Shape shape = (Shape) shapeClass.newInstance();
    shape.printShape();
  }
}
```

* reflection을 이용하면 프로그램 실행 이후 필요한 클래스를 선택해서 사용할 수 있을 것이다. 