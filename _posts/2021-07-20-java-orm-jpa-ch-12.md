---
title: "JPA 뽀개기 - 스프링 데이터 JPA"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 12. 스프링 데이터 JPA

#### 스프링 데이터 JPA 소개

* 스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 돕는 프로젝트
* CRUD 위한 공통 인터페이스 제공
* 레포지토리 개발 시 인터페이스만 작성하면 실행 시점에 구현 객체 동적 생성 및 주입

##### 스프링 데이터 프로젝트

* 다양한 데이터 저장소에 대한 접근을 추상화
* 스프링 데이터 JPA 프로젝트는 JPA 특화



#### 공통 인터페이스 기능

* `JpaRepository` 를 상속받고 제네릭에 엔티티 클래스와 엔티티 클래스가 사용하는 식별자 타입을 지정하는 게 스프링 데이터 JPA를 사용하는 가장 단순한 방법
* 조회, 저장, 삭제 등의 기본적인 메소드를 제공



#### 쿼리 메소드 기능

##### 메소드 이름으로 쿼리 생성

* 정해진 규칙에 따라 메소드 이름을 지으면 메소드 이름을 분석해 JPQL을 생성하고 실행
* 엔티티의 필드명이 변경되면 인터페이스에 정의된 메소드 이름도 꼭 함께 변경해야 한다.

##### JPA NamedQuery

* 메소드 이름으로 JPA Named 쿼리를 호출하는 기능
* 애노테이션이나 XML에 쿼리 정의 후 정의한 `Named` 쿼리를 메소드 이름만으로 호출 가능

````java
@Entity
@NamedQuery(
	name="Member.findByUsername",
  query="select m from Member m where m.username =:username"
)
public class Member {
  ...
}
---
public interface MemberRepository extends JpaRepository<Member, Long> {
  List<Member> findByUsername(@Param("username") String username);
}
````

##### @Query, 레포지토리 메소드에 쿼리 정의

* `@Query` 어노테이션을 통해 레포지토리 메소드에 직접 쿼리를 정의할 수 있다.
* 실행할 메소드에 정적 쿼리를 직접 작성하므로 이름 없는 `Named` 쿼리라고 할 수 있다. 
* 애플리케이션 실행 시험에 문법 오류를 발견할 수 있다.

##### 파라미터 바인딩

* 위치 기반: 파라미터 순서로 바인딩하며 1부터 시작(네이티브 SQL은 0부터 시작)
* 이름 기반

##### 벌크성 수정 쿼리

* 벌크성 수정, 삭제 쿼리는 `org.springframework.data.jpa.repository.Modifying` 어노테이션을 사용하면 된다.

##### 반환 타입

* 결과가 한 건 이상이면 컬렉션 인터페이스를 사용하고 단건이면 반환 타입을 지정한다.
* 결과가 없으면 컬렉션은 빈 컬렉션, 단건은 `null` 을 반환하며 단건을 기대하고 반환 타입을 지정했는데 결과가 2건 이상 조회되면 예외가 발생한다.
  * 단건으로 지정한 메소드를 호출하면 내부에서 `Query.getSingleResult()` 를 호출하는데 이 메소드는 조회 결과가 없으면 예외가 발생한다. 이 예외는 다루기 불편해 스프링 데이터 JPA는 이 예외 대신 `null` 을 반환한다.

##### 페이징과 겅

* 페이징과 정렬 기능을 사용할 수 있도록 `Sort`, `Pageable` 두 가지의 특별한 파라미터를 제공한다.

##### 힌트

* `org.springframework.data.jpa.repository.QueryHints` 어노테이션을 사용하면 된다. SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트다.

##### Lock

* `org.springframework.data.jpa.repository.Lock` 어노테이션 사용



#### 명세

* 명세를 이해하기 위한 핵심 단어는 술어(*predicate*)인데 이것은 참이나 거짓으로 평가된다. 그리고 AND, OR 같은 연산자로 조합할 수 있다. 데이터를 검색하기 위한 제약 조건 하나하나를 술어라고 할 수 있다.
* 이 술어를 스프링 데이터 JPA는 `org.springframework.data.jpa.domain.Specitication` 클래스로 정의했다.
  * 컴포지트 패턴으로 구성되어 있어 여러 `Specification` 을 조합할 수 있다. 다양한 검색 조건을 조립해 새로운 검색 조건을 쉽게 만들 수 있다.
* 명세 기능을 사용하려면 `JpaSpecificationExecutor` 인터페이스를 상속받으면 된다.
  * `Specification` 을 파라미터로 받아서 검색 조건으로 사용한다.
* 명세를 정의하려면 `Specification` 인터페이스를 구현하면 된다.
  * 명세를 정의할 때는 `toPredicate` 메소드만 구현다면 되는데 JPA Criteria의 `Root`, `CriteriaQuery`, `CriteriaBuilder` 클래스가 모두 파라미터로 주어진다. 이들을 활용해 적절한 검색 조건을 반환하면 된다.



#### 사용자 정의 레포지토리 구현

* 먼저 사용자 정의 인터페이스를 작성한다. 이름은 자유롭게 작성할 수 있다.

* 레포지토리 인터페이스 이름 + `Impl`로 사용자 정의 인터페이스를 구현한 클래스를 작성하면 스프링 데이터 JPA가 사용자 정의 구현 클래스로 인식한다.
* 레포지토리 인터페이스에 사용자 정의 인터페이스를 상속받으면 된다.



#### Web 확장

* `SpringDataWebConfiguration` 을 스프링 빈으로 등록하고 `@EnableSpringDataWebSupport` 를 설정해 주면 도메인 클래스 컨버터와 페이징을 위한 리졸버가 스프링 빈으로 등록된다.

##### 도메인 클래스 컨버터 기능

* HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩해 준다.
* 도메인 클래스 컨버터는 해당 엔티티와 관련된 레포지포리를 사용해서 엔티티를 찾는다.

##### 페이징과 정렬 기능

* `HandlerMethodArgumentResolver` 를 제공해 페이징과 정렬 기능을 스프링 MVC에서 편리하게 사용할 수 있다.



#### 스프링 데이터 JPA가 사용하는 구현체

* `@Repository`: JPA 예외를 스프링이 추상화한 예외로 변환
* `@Transactional`: JPA의 모든 변경은 트랜잭션 안에서 이루어져야 한다. 스프링 데이터 JPA가 제공하는 공통 인터페이스를 사용하면 데이터를 변경하는 메소드에 이 어노테이션으로 트랜잭션 처리가 되어 있다. 서비스 계층에서 트랜잭션을 시작하지 않으면 레포지토리에서 시작하고, 서비스 계층에서 시작했으면 해당 트랜잭션을 전파받아 사용한다.



#### 스프링 데이터 JPA와 QueryDSL 통합

##### `QueryDslPredicateExecutor` 사용

* 레포지토리에서 상속받는다. 
  * QueryDSL을 검색 조건으로 사용하면서 스프링 데이터 JPA가 제공하는 페이징과 정렬 기능도 함께 사용할 수 있다.
  * `join`, `fetch`는 사용할 수 없으므로 더 다양한 기능을 사용하려면 `JPAQuery` 를 직접 사용하거나 스프링 데이터 JPA가 제공하는 `QueryDslRepositorySupport` 를 사용해야 한다.

##### `QueryDslRepositorySupport` 사용

* 사용자 정의 레포지토리 인터페이스를 만들고 구현체를 작성한다.