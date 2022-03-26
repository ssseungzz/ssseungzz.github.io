---
title: "Modern Java in Action 13장; 디폴트 메서드"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 13. 디폴트 메서드
* 자바 8에서는 기본 구현을 포함하는 인터페이스를 정의하는 두 가지 방법을 제공한다.
  * 인터페이스 내부에 정적 메서드 사용
  * 인터페이스의 기본 구현을 제공할 수 있도록 디폴트 메서드 기능 사용
* `default` 키워드는 해당 메서드가 디폴드 메서드임을 나타낸다.
* 디폴트 메서드를 사용하면 자바 API의 호환성을 유지하면서 라이브러리를 바꿀 수 있다.
* 자바에는 인터페이스 그리고 인터페이스의 인스턴스를 활용할 수 있는 다양한 정적 메서드를 정의하는 유틸리티 클래스를 활용한다. 
  * 자바 8부터는 인터페이스에 직접 정적 메서드를 선언할 수 있으므로 유틸리티 클래스를 없애고 직접 인터페이스 내부에 정적 메서드를 구현할 수 있으나 과거 버전과의 호환성을 위해 자바 API에는 유틸리티 클래스가 남아 있다.

#### 13.1 변화하는 API
###### 사용자가 겪는 문제
* 인터페이스에 메서드를 추가하면 바이너리 호환성은 유지된다.
  * 바이너리 호환성이란 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현 없이도 기존 클래스 파일 구현이 잘 동작한다는 의미다.
  * 그러나 코드가 바뀌면 런타임 에러가 발생할 수 있다.
* 전체 애플리케이션을 재빌드할 때 컴파일 에러가 발생한다.
  * 공개된 API를 고치면 기존 버전과의 호환성 문제가 발생한다.
* 디폴트 메서드를 사용하면 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다.

#### 13.2 디폴트 메서드란 무엇인가?
* 자바 8에서는 호환성을 유지하면서 API를 바꿀 수 있도록 디폴트 메서드를 제공한다.
* 인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본으로 제공한다.
* `default`라는 키워드로 시작하며 다른 클래스에 선언된 메서드처럼 메서드 바디를 포함한다.
> 추상 클래스와 인터페이스 모두 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있다. 차이는 클래스는 하나의 추상 클래스만 상속받을 수 있지만 인터페이스를 여러 개 구현할 수 있다는 것, 그리고 추상 클래스는 인스턴스 변수로 공통 상태를 가질 수 있다는 것이다.

#### 13.3 디폴트 메서드 활용 패턴
##### 13.3.1 선택형 메서드
* 기본 구형을 제공해서 인터페이스를 구현하는 클래스는 빈 메서드를 구현할 필요가 없어졌다.

##### 13.3.2 동작 다중 상속
###### 다중 상속 형식
* 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작(구현 코드)을 상속받을 수 있다.
###### 기능이 중복되지 않는 최소의 인터페이스
* 인터페이스를 기능으로 나누고 기본 구현을 제공할 수 있다.
  * 이들 인터페이스를 조합해서 필요에 따라 다양한 클래스를 구현할 수 있다.
  * 예를 들어 `Moveable`, `Rotatable`, `Resizable` 등의 인터페이스에 각각 기본 구현을 제공해서 움직일 수 있고 회전할 수 있지만 크기 조절은 불가능한 클래스, 움직일 수 없지만 회전할 수 있는 클래스 등 다양한 조합을 만들어낼 수 있다.
> 상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다. 가령, 한 개의 메서드를 재사용하기 위해 100개의 메서드와 필드가 있는 클래스를 상속받는 것은 좋은 생각이 아니다. 이럴 때는 델리게이션, 즉 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다.

#### 13.4 해석 규칙
##### 13.4.1 알아야 할 세 가지 해결 규칙
* 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
* 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다. 즉, B가 A를 상속받는다면 B가 A를 이긴다.
* 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.
* 위 규칙(클래스와 메서드 관계)으로 디폴트 메서드를 선택할 수 없는 상황에서는 직접 사용하려는 메서드를 명시적으로 선택해야 한다.
```java
public class C implements B, A {
  void hello() {
    B.super.hello(); // 명시적으로 B의 메서드 선택
  }
}
```