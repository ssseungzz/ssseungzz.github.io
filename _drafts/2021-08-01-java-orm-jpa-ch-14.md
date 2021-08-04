---
title: "JPA 뽀개기 - 컬렉션과 부가 기능"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 14. 컬렉션과 부가 기능

#### 컬렉션

* JPA는 자바에서 기본으로 제공하는 `Collection`, `List`, `Set`, `Map` 컬렉션을 지원하고 다음 경우에 이 컬렉션을 사용할 수 있다.
  * `@OneToMany`, `@ManyToMany`를 사용해서 일대다나 다대다 엔티티를 매핑할 때
  * `@ElementCollection` 을 사용해서 값 타입을 하나 이상 보관할 때

![](https://gblobscdn.gitbook.com/assets%2F-M5HOStxvx-Jr0fqZhyW%2F-MACmDoRtjrcVydLyXZ4%2F-MACmUlIDmbXLugEnIqa%2F1.png?alt=media&token=66a09758-74ca-4d8a-acd3-9b72f82390f7)

* `Collection`: 자바가 제공하는 최상위 컬렉션으로 하이버네이트는 중복을 허용하고 순서를 보장하지 않는다고 가정한다.
* `Set`: 중복을 허용하지 않는 컬렉션이다. 순서를 보장하지 않는다.
* `List`: 순서를 보장하고 중복을 허용한다.
* `Map`: `Key`, `Value` 구조로 되어 있는 컬렉션이다.

##### JPA와 컬렉션

* 하이버네이트는 엔티티를 영속 상태로 만들 때 컬렉션 필드를 하이버네이트에서 준비한 컬렉션으로 감싸서 사용한다.

```java
@Entity
public class Team {
  
  @Id
  private String id;
  
  @OneToMany
  @JoinColumn
  private Collection<Member> members = new ArrayList<Member>();
  // ...
}
```

* 위의 엔티티를 영속 상태로 만들고 `members` 필드의 클래스를 출력하면 아래와 같다.

```java
members: class.org.hibernate.collection.internal.PersistentBag
```

* 하이버네이트는 컬렉션을 효율적으로 관리하기 위해 엔티티를 영속 상태로 만들 때 원본 컬렉션을 감싸고 있는 내장 컬렉션을 생성해서 이 내장 컬렉션을 사용하도록 참조를 변경한다. 원본 컬렉션을 감싸고 있어 래퍼 컬렉션으로도 부른다.
* 인터페이스마다 다른 래퍼 컬렉션을 사용한다.

##### Collection, List

* 중복을 허용하는 컬렉션이고 `PersistentBag`을 래퍼 컬렉션으로 사용한다.
  * 중복을 허용하고 순서를 보관하지 않는다.
* `ArrayList`로 초기화하면 된다.
* 엔티티를 추가할 때 중복된 엔티티가 있는지 비교하지 않고 저장만 하면 된다. 따라서 엔티티를 추가해도 지연 로딩된 컬렉션을 초기화하지 않는다.

##### Set

* 중복을 허용하지 않는다. `PersistentSet`을 컬렉션 래퍼로 사용한다.
* `HashSet`으로 초기화하면 된다.
  * 중복을 허용하지 않으므로 객체를 추가할 때마다 `equals()` 메소드로 같은 객체가 있는지 비교한다. 같은 객체가 없으면 추가하고 `true`를 반환하고, 같은 객체가 있어 추가에 실패하면 `false`를 반환한다.
* 엔티티를 추가할 때 중복된 엔티티가 있는지 비교해야 한다. 따라서 엔티티를 추가할 떄 지연 로딩된 컬렉션을 초기화한다.

##### List + `@OrderColumn`

* 순서가 있는 특수한 컬렉션으로 인식한다. 순서가 있다는 의미는 데이터베이스에 순서 값을 저장해서 조회할 때 사용한다는 의미이다. `PersistentList`를 사용한다.

```java
@Entity
public class Board {
   
  @Id @GeneratedValue
  private Long id;
  
  private String title;
  private String content;
  
  @OneToMany(mappedBy = "board")
  @OrderColumn(name = "POSITION")
  private List<Comment> comments = new ArrayList<Comment>();
  
  //...
}
```

* 순서가 있는 컬렉션은 데이터베이스에 순서 값도 함께 관리한다.
* 위의 예시에서는 `@OrderColumn`의 `name` 속성에 `POSITION` 이라는 값을 주었다. JPA는 위치 값을 테이블의 `POSITION` 컬럼에 보관한다. 
  * 그런데 `Board.comments` 컬렉션은 `Board` 엔티티에 있지만 테이블의 일대다 관계의 특성상 위치 값은 다쪽에 저장해야 한다. 따라서 실제 `POSITION` 컬럼은 `COMMENT` 테이블에 매핑된다.
* `@OrderColumn`은 몇 가지 단점 때문에 실무에서 잘 사용하지 않는다.
  * `@OrderColumn` 을 `Board` 엔티티에서 매핑하므로 `Comment` 에서는 `POSITION` 값을 알 수 없다. 그래서 `Comment`를 INSERT 시 `POSITION` 값이 저장되지 않는다. `POSITION`은 `Board.comments`의 위치 값이므로 이 값을 사용해서 `POSITION`의 값을 UPDATE 하는 SQL이 추가로 발생한다.
  * `List`를 변경하면 연관된 많은 위치 값을 변경해야 한다. 예시에서 댓글 하나를 삭제하면 다른 댓글의 `POSITION` 값을 수정하는 SQL이 추가로 실행된다.
  * 중간에 `POSITION` 값이 없으면 조회한 `List` 에는 `null` 값이 보관된다. 특정 댓글을 데이터베이스에서 강제로 삭제하고 다른 댓글들의 `POSITION` 값을 수정하지 않으면  `List`에서 해당 위치에 `null` 값이 보관된다.

##### `@OrderBy`

* 데이터베이스의 ORDER BY 절을 사용해서 컬렉션을 정렬한다.
  * 순서용 컬럼을 매핑하지 않아도 된다.
  * 모든 컬렉션에 사용 가능하다.
  * JPQL의 `order by` 절처럼 엔티티의 필드를 대상으로 한다.



#### `@Converter`

* 엔티티의 데이터를 변환해서 데이터베이스에 저장할 수 있다.
* 예를 들어 회원의 VIP 여부를 자바의 `boolean` 타입으로 사용하고 싶다고 하자. JPA를 사용하면 데이터베이스에 저장될 때 보통 0 또는 1인 숫자로 저장된다. 그런데 데이터베이스에 숫자 대신 문자 Y 또는 N으로 저장하고 싶다면 컨버터를 사용하면 된다.

```java
@Entity
public class Member {
  // ...
  
  @Conver(converter=BooleanToYNConverter.class)
  private boolean vip;
  
  //...
}
```

* `@Convert`를 적용해서 데이터베이스에 저장되기 직전에 `BooleanToYNConverter`가 동작한다.

```java
@Converter
public clas BooleanToYNConverter implements AttributeConverter<Boolean, String> {
  
  @Override
  public String convertToDatabaseColumn(Boolean attribute) {
    return (attribute != null && attribute) ? "Y" : "N";
  }
  
  @Override
  public Boolean convertTonEntityAttribute(String dbData) {
    return "Y".equals(dbData);
  }
}
```

* 컨버터 클래스는 `@Converter` 어노테이션을 사용하고 `AttributeConverter` 인터페이스를 구현해야 한다. 그리고 제네릭에 현재 타입과 변환할 타입을 지정해야 한다.
  * 구현해야 할 두 메소드는 각각 엔티티의 데이터를 데이터베이스 컬럼에 저장할 데이터로 변환하는 메소드와 데이터베이스에서 조회한 컬럼 데이터를 엔티티의 데이터로 변환하는 메소드이다.
* 컨버터는 클래스 레벨에도 설정할 수 있는데 이때는 `attributeName` 속성을 사용해 어떤 필드에 컨버터를 적용할지 명시해야 한다.

##### 글로벌 설정

* `@Converter(autoApply = true)` 옵션으로 해당하는 모든 타입에 턴버터를 적용할 수 있다.



#### 리스너

* JPA 리스터 기능을 사용하면 엔티티의 생명주기에 따른 이벤트를 처리할 수 있다.

##### 이벤트 종류

> 이벤트의 종류와 발생 시점은 아래와 같다.

![](https://leejaedoo.github.io/assets/img/listener.jpeg)

1. `PostLoad`: 엔티티가 영속성 컨텍스트에 조회된 직후 또는 `refresh`를 호출한 후
2. `PrePresist`: `persist()` 메소드를 호출해서 엔티티를 영속성 컨텍스트에 관리하기 직전에 호출된다. 식별자 생성 전략을 사용한 경우 엔티티에 식별자는 아직 존재하지 않는다. 새로운 인스턴스를 `merge` 할 때도 수행된다.
3. `PreUpdate`: `flush`나 `commit`을 호출해서 엔티티를 데이터베이스에 호출하기 직전에 호출된다. 
4. `PreRemove`: `remove()` 메소드를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출된다. 또한 삭제 명령어로 영속성 전이가 일어날 때도 호출된다. 
5. `PostPersist`: `flush`나 `commit`을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출된다. 식별자가 항상 존재한다. 식별자 생성 전략이 `IDENTITY`면 식별자를 생성하기 위해 `persist()` 를 호출하면서 데이터베이스에 해당 엔티티를 저장하므로 이때는 `persist()`를 호출한 직후에 호출된다.
6. `PostUpate`: `flush`나 `commit`을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출된다.
7. `PostRemove`: `flush`나 `commit`을 호출해서 엔티티를 데이터베이스에서 삭제한 직후에 호출된다.

##### 이벤트 적용 위치

* 엔티티에 직접 적용
  * 엔티티에 이벤트가 발생할 때마다 어노테이션으로 지정한 메소드가 실행된다.

```java
@Slf4j
@Entity
public class Cat {
  
  @Id @GeneratedValue
  public Long id;
  
  private String name;
  
  @PrePersist
  public void prePresist() {
    log.info("Cat.prePersist id=" + id);
  }
  
  @PostPersist
  public void postPersist() {
    // ...
  }
  
  // ...
}
```

* 별도의 리스너 등록
  * 리스너는 대상 엔티티를 파라미터로 받을 수 있다. 반환 타입은 `void`로 설정해야 한다.

```java
@Entity
@EntityListeners(CatListener.class)
public class Cat {
  // ...
}

public class CatListener {
  
  @PrePersist
  private void prePersist(Object obj) {
    // ...
  }
  
  @PostPersist
  private void postPersist(Cat cat) { // 특정 타입이 확실하면 특정 타입을 받을 수 있다.
    // ...
  }
}
```

* 기본 리스터 사용
  * `META-INF/orm.xml` 에 기본 리스너로 리스너 등록
* 여러 리스너 등록 시 이벤트 호출 순서
  1. 기본 리스너
  2. 부모 클래스 리스너
  3. 리스너
  4. 엔티티
* 더 세밀한 설정
  * `@ExcludeDefaultListeners`: 기본 리스너 무시
  * `@ExcludeSuperclassListeners`: 상위 클래스 이벤트 리스너 무시



#### 엔티티 그래프

* 엔티티를 조회할 때 연관된 엔티티를 함께 조회하려면?
  * 글로벌 `fetch` 옵션을 `FetchType.EAGER`로 설정
  * JPQL에서 페치 조인 사용 - `select o from Order o join fetch o.member`
* 글로벌 옵션은 애플리케이션 전체에 영향을 주고 변경할 수 없으므로 글로벌 `fetch` 옵션은 주로 `FetchType.LAZY`로 설정하고 엔티티를 조회할 때 연관된 엔티티를 함께 조회할 필요가 있으면 JPQL의 페치 조인을 사용한다.
* 그런데 페치 조인을 사용하면 같은 JPQL을 중복해서 작성하는 경우가 많다.
  * `select o from Order o where o.status = ?`
  * `select o from Order o join fetch o.member where o.status = ?`
  * `select o from Order o join fetch o.orderItems where o.status = ?`
  * 위의 세 가지 모두 주문을 조회하는 같은 JPQL이지만 함께 조회할 엔티티에 따라 다른 JPQL을 사용해야 한다. JPQL이 데이터를 조회하는 기능뿐 아니라 연관된 엔티티를 함께 조회하는 기능도 제공하기 때문이다.
* 엔티티 그래프 기능을 사요하면 엔티티를 조회하는 시점에 함께 조회할 엔티티를 선택할 수 있다.
  * 정적으로 정의하는 `Named` 엔티티 그래프와 동적으로 정의하는 엔티티 그래프가 있다.

##### Named 엔티티 그래프

