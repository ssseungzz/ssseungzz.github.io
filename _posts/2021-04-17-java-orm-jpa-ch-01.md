---
title: "JPA 뽀개기 - JPA 소개"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 1. JPA 소개

>  애플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점은 무엇일까?

* 진정한 의미의 계층 분할이 어렵다.
  * SQL에 모든 것을 의존하는 상황에서는 DAO를 열어 어떤 SQL이 실행되고 어떤 객체들이 함께 조회되는지 알아야 한다.
* 계층 분할이 어렵기 때문에 엔티티를 신뢰할 수 없다.
* SQL에 의존적인 개발을 하게 된다.

> JPA는 이런 문제를 어떻게 해결할까?

* JPA를 사용하면 객체를 데이터베이스에 저장하고 관리할 때 개발자가 SQL을 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다. 그러면 JPA가 개발자 대신 적절한 SQL을 생성한다.

> 객체 지향과 관계형 데이터베이스의 패러다임의 차이에서 오는 또 다른 문제

* 객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 있는 참조에 접근해서 연관된 객체를 조회한다. 반면에 테이블은 외래 키를 사용해서 다른 테이블과 연관관계를 가지고 조인을 사용해서 연관된 테이블을 조회한다.
  * 객체는 참조가 있는 방향으로만 조회할 수 있다. 반면 테이블은 외래 키 하나로 원하는 방향의 조회가 가능하다.
  *  객체는 참조로 연관관계를 맺고, 테이블은 외래 키로 연관관계를 맺기 때문에 개발자가 중간에서 변환 역할을 수행해야 한다.
* JPA는 연관관계가 설정된 객체의 참조를 외래 키로 변환해서 적절한 SQL을 데이터베이스에 전달한다. 객체를 조회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해 준다.
* 참조를 타고 객체와 연관관계가 있는 다른 객체를 찾는 것을 객체 그래프 탐색이라고 한다.
  * SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다. 
  * 객체와 연관된 모든 다른 객체들을 데이터베이스에서 조회해서 애플리케이션 메모리에 올려 둘 수는 없으므로 상황에 맞는 메서드를 여러 개 만들어 두어야 한다.
  * JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다. (**지연 로딩**) 따라서 연관된 객체를 신뢰하고 마음껏 조회할 수 있다.
* JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다.

#### JPA는 무엇일까?

* JPA(Java Persistence API)는 자바 진영의 ORM 기술 표준이다. ORM은 이름 그대로 객체와 관계혀 데이터베이스를 매핑한다는 의미이다.
  * ORM 프레임워크에는 다양한 종류가 존재한다. 하이버네이트가 대표적.
  * JPA는 자바 ORM 기술에 대한 API 표준 명세다.