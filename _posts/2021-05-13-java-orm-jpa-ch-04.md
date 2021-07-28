---
title: "JPA 뽀개기 - 엔티티 매핑"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 4. 엔티티 매핑

#### @Entity

* JPA를 사용해서 테이블과 매핑할 클래스는 이 어노테이션을 필수로 붙여야 한다.
* 이 어노테이션이 붙은 클래스는 JPA가 관리하는 것이다.
* 속성에는 이름(`name`)이 있다. 설정하지 않으면 클래스 이름을 그대로 사용한다.
* **기본 생성자는 필수**이며 `final`, `enum`, `interface`, `inner` 클래스에는 사용할 수 없다. 
* 저장할 필드에 `final` 을 사용하면 안 된다.



#### @Table

* 엔티티와 매핑할 테이블을 지정한다. 생략하면 매핑한 엔티티 이름을 테이블 이름으로 사용한다
* 이름(`name`). 카탈로그(`catalog`), 스키마(`schema`), 유니크 제약 조건(`uniqueConstraints`) 등을 속성으로 가진다.



#### 다양한 매핑 사용

* 자바의 `enum`  을  필드로 사용하려면 `@Enumerated` 어노테이션으로 매핑한다.
* 자바의 날짜 타입은 `@Temporal`을 사용해서 매핑한다.
* 길이 제한이 없는 문자열 필드의 경우 `@Lob`을 사용해서 CLOB, BLOB 타입을 매핑할 수 있다.




#### 데이터베이스 스키마 자동 생성

* 클래스의 매핑 정보를 보면 어떤 테이블에 어떤 컬럼을 사용하는지 알 수 있다. JPA는 이 매핑 정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 생성한다.
* `persistence.xml`에 `<property name="hibernate.hbm2ddl.auto" value="create" />` 속성을 추가하면 애플리케이션 실행 시점에 데이터베이스 테이블을 **자동으로 생성**한다. 자동 생성되는 DDL은 지정한 데이터베이스 방언에 따라 달라진다.  
* 스키마 자동 생성 기능은 개발자가 테이블을 직접 생성하는 수고를 덜 수 있지만 운영 환경에서 사용할 만큼 완벽하지는 않으므로 개발 환경에서 사용학나 매핑을 어떻게 해야 하는지 참고하는 정도로만 사용하는 것이 좋다.
* 속성에는 `create` 외에도 `create-drop`(애플리케이션 종료 시 생성한 DDL 삭제), `update`(데이터베이스 테이블과 엔티티 매핑 정보를 비교해 변경 사항만 수정), `validate`(데이터베이스 테이블과 엔티티 매핑 정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행 X), `none` 등이 있다.
  * 자바는 카멜 표기법을 사용하고 데이터베이스는 언더스코어를 주로 사용한다. 이에 맞게 매핑하려면 `@Column.name` 속성을 명시적으로 사용해서 이름을 지어 주어야 한다.



#### DDL 생성 기능

* `@Column`에 `nullable`, `length` 등의 속성 값을 지정하면 자동 생성되는 DDL에 해당 조건이 추가된다. `@Table`의 경우도 마찬가지로 `uniqueConstraints` 등의 제약 조건을 추가하면 DDL에 해당 내용이 반영된다.
* 이런 기능들은 단지 **DDL을 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.



#### 기본 키 매핑

* JPA는 다양한 데이터베이스 기본 키 생성 전략을 제공한다. 데이터베이스 벤더마다 지원하는 방식이 다르기 때문이다. 

##### 직접 할당

* 기본 키를 직접 할당하려면 `@Id`를 매핑하면 된다.
  * 자바 기본형, 래퍼형, `String`, `java.util.Date`, `java.sql.Date`, `java.math.BigDecimal`, `java.math.BigInteger` 등에 매핑 가능하다.
  * 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당한다.

##### IDENTITY 전략

* 기본 키 생성을 데이터베이스에 위임하는 전략
* 데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용한다.
* 개발자가 엔티티에 직접 식별자를 할당하면 `@Id` 어노테이션만 있으면 되지만 식별자가 생성되는 경우에는 `@GeneratedValue` 어노테이션을 사용하고 식별자 생성 전략을 선택해야 한다. `IDENTITY` 전략을 사용하려면 `@GeneratedValue(strategy = GenerationType.IDENTITY`)로 어노테이션과 속성 값을 지정하면 된다.
* 이 전략을 사용하면 JPA는 기본 키 값을 얻어 오기 위해 데이터베이스를 추가로 조회한다.
  * `Statement.getGeneratedKeys()`를 사용하면 데이터를 저장하면서 동시에 생성된 기본 키 값도 얻어 올 수 있다. 하이버네이트는 이 메서드를 사용해서 데이터베이스와 한 번만 통신한다.
* 엔티티가 영속 상태가 되려면 식별자가 반드시 필요한데 이 전략의 경우 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 `persist()` 호출 즉시 INSERT SQL이 전달된다. 따라서 트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다.

##### SEQUENCE 전략

* 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트인 데이터베이스 시퀀스를 사용해서 기본 키를 생성한다. 사용할 데이터베이스 시퀀스를 생성해 두고 해당 시퀀스를 매핑해 준다.

```java
@Entity
@SequenceGenerator(
	name = "BOARD_SEQ_GENERATOR", // 사용할 시퀀스 생성기의 이름(실제 데이터베이스 시퀀스와 매핑)
  sequenceName = "BOARD_SEQ", // 매핑할 실제 데이터베이스 시퀀스 이름
  initialValue = 1, allocationSize = 1
)
public class Board {
  
  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,
                  generator = "BOARD_SEQ_GENERATOR")
  private Long id;
}
```

* `persist()` 호출 시 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회하고 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 할다앟ㄴ다. 이후 **트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장**한다 (IDENTITY 전략은 엔티티를 **먼저 데이터베이스에 저장**한 후에 식별자를 조회)
* `@SequenceGenerator`의 속성에는 `name`, `sequenceName`, `initialValue`, `allocationSize`, `catalog`(데이터베이스  카탈로그 이름), `schema`(데이터베이스 스키마 이름)가 있다.
  * 매핑할 DDL은 `create sequence [sequenceName] start with [initialValue] increment by [allocationSize]`
  * 속성 기본 값은 JPA 구현체가 정의한다.
  * JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 `@SequenceGenerator.allocationSize`를 사용한다. 설정한 값만큼 한 번에 시퀀스 값을 증가시키고 나서 그만큼에 메모리에 시퀀스 값을 할당한다. 이 방법은 시퀀스 값을 선점하므로 여러 JVM이 동시에 동작해도 기본 키 값이 충돌하지 않는다는 장점이 있지만 데이터베이스에 직접 접근해서 데이터를 등록할 때 시퀀스 값이 한 번에 많이 증가한다는 점을 주의해야 한다.

##### TABLE 전략

* 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다. 테이블을 사용하므로 모든 데이터베이스에 적용 가능하다.
* 이 전략을 사용하기 위해서는 키 생성 용도로 사용할 테이블을 만들어야 한다. 특정 컬럼을 시퀀스 이름으로, 특정 컬럼을 시퀀스 값으로 사용한다. `sequence_name` 컬럼을 시퀀스 이름으로, `next_val` 컬럼을 시퀀스 값으로 사용하는 것을 기본으로 한다.

```java
@Entity
@TableGenerator(
	name = "BOARD_SEQ_GENERATOR", // 테이블 키 생성기의 이름
  table = "MY_SEQUENCES", // 매핑할 키 생성용 테이블(데이터베이스 테이블)
  pkColumnValue = "BOARD_SEQ", allocationSize = 1
)
public class Board {
  
  @Id
  @GeneratedValue(strategy = GenerationType.TABLE,
                  generator = "BOARD_SEQ_GENERATOR") // 테이블 키 생성기 지정
  private Long id;
}
```

* `@TableGenerator`를 사용해서 테이블 키 생성기를 등록한다. `@TableGenerator.pkColumnValue`에서 지정한 컬럼명이 데이터베이스 키 테이블에 추가된다. 키 생성기를 사용할 때마다 `next_val` 컬럼 값이 증가한다.
*  `@TableGenerator`는 `name`, `table`, `pkColumn`(시퀀스 컬럼명, 기본 값 `sequence_name`), `valueColumnName`(시퀀스 값 컬럼명, 기본 값 `next_val`), `pkColumnValue`(키로 사용할 값 이름, 기본 값 엔티티 이름) 등의 속성이 있다.
* TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고 다음 값으로 증가시키기 위해 UPDATE 쿼리를 사용한다. 이 전략은 데이터베이스와 한 번 더 통신하는 단점이 있다.

##### AUTO 전략

* 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다. 
* SEQUENCE나 TABLE 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어 두어야 한다. 만약 스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본 값을 사용해서 적절한 시퀀스나 키 생성용 테이블을 만들어 준다.

###### 권장하는 식별자 선택 전략

데이터베이스 기본 키는 아래의 3가지 조건을 만족해야 한다.

1. null 값은 허용하지 않는다.
2. 유일해야 한다.
3. 변해선 안 된다.

테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.

* 자연 키
  * 비즈니스에 의미가 있는 키
  * 예: 주민등록번호, 이메일, 전화번호
* 대리 키
  * 비즈니스와 관련 없는 임의로 만들어진 키
  * 예: 오라클 시퀀스, `auto_increment`, 키 생성 테이블 사용

자연 키보다는 대리 키

* 자연 키인 전화번호를 기본 키로 선택하면 그 번호가 유일할 수는 있지만 누군가는 전화번호가 없을 수 있고 누군가의 전화번호는 변경될 수 있다. 따라서 적당하지 않다.

* 변하지 않을 것 같은 자연 키도 비즈니스 환경에 의해 얼마든지 변화할 수 있으므로 비즈니스와 관련 없는 대리 키가 낫다. 비즈니스 요구 사항은 계속해서 변하는 데에 비해 테이블은 한번 정의하면 변경하기 어렵다.



##### 필드와 컬럼 매핑

* `@Column`을 생략하게 되면 `@Column` 속성의 기본 값이 대부분 적용된다. 그런데 자바 기본 타입은 null 값을 입력할 수 없고, 객체 타입일 경우에만 null 값이 허용된다. 따라서 자바 기본 타입 필드를 DDL로 생성할 때는 not null 제약 조건을 추가하는 것이 안전하다.
* JPA는 이를 고려해서 DDL 생성 기능 사용 시 기본 타입에는 not null 조건을 추가한다. 다만 기본 타입에 `@Column` 을 사용하면 `nullable = true`가 기본 값이므로 not null 제약 조건을 설정하지 않게 된다. 따라서 자바 기본 타입에 `@Column` 사용 시 `nullable = false`로 지정하는 게 안전하다. 