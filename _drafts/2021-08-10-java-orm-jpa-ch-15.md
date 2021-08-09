---
title: "JPA 뽀개기 - 고급 주제와 성능 최적화"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 15. 고급 주제와 성능 최적화

#### 예외 처리

##### JPA 표준 예외 정리

* `javax.persistence.PersistenceException`의 자식 클래스인데 이 예외 클래스는 `RuntimeException`의 자식이다.
* 크게 2가지로 나눌 수 있다.

1. 트랜잭션 롤백을 표시하는 예외

* 심각한 예외이므로 복구해선 안 된다. 이 예외가 발생하면 트랜잭션을 강제로 커밋해도 트랜잭션이 커밋되지 않고 `RollbackException` 예외가 발생한다.

| 트랜잭션 롤백을 표시하는 예외  | 설명                                                         |
| :----------------------------- | ------------------------------------------------------------ |
| `EntityExistsException`        | `em.persist()` 호출 시 이미 같은 엔티티가 있으면 발생        |
| `EntityNotFoundException`      | `em.getReference()` 호출했는데 실제 사용 시 엔티티가 존재하지 않으면 발생 |
| `OptimisticLockException`      | 낙관적 락 충돌 시 발생                                       |
| `PessimisticLockException`     | 비관적 락 충돌 시 발생                                       |
| `RollbackException`            | 커밋 실패 시 발생, 롤백이 표시되어 있는 트랜잭션 커밋 시에도 발생 |
| `TransactionRequiredException` | 트랜잭션이 필요할 때 트랜잭션이 없으면 발생, 트랜잭션 없이 엔티티를 변경할 때 주로 발생 |

2. 트랜잭션 롤백을 표시하지 않는 예외

* 심각한 예외가 아니다. 개발자가 트랜잭션을 커밋할지 롤백할지 판단하면 된다.

| 트랜잭션 롤백을 표시하지 않는 예외 | 설명                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `NoResultException`                | `Query.getSingleResult()` 호출 시 결과가 하나도 없을 때 발생 |
| `NonUniqueResultException`         | `Query.getSingleResult()` 호출 시 결과가 둘 이상일 때 발생   |
| `LockTimeoutException`             | 비관적 락에서 시간 초과 시 발생                              |
| `QueryTimeoutException`            | 쿼리 실행 시간 초과 시 발생                                  |

##### 스프링 프레임워크의 JPA 예외 반환

* 데이터 접근 게층의 구현 기술에 직접 의존하는 것은 좋은 설계라 할 수 없다.
* 스프링은 데이터 접근 계층에 대한 예외를 추상화해서 개발자에게 제공한다.
* JPA 표준 명세상 발생할 수 있는 예외도 추상화해서 제공한다.

##### 스프링 프레임워크에 JPA 예외 변환기 적용

* JPA 예외를 스프링이 제공하는 추상화된 예외로 변경하려면 `PersistenceExceptionTranslationPostProcessor`를 스프링 빈으로 등록하면 된다.
  * `@Repository` 어노테이션을 사용한 곳에 예외 변환 AOP를 적용해서 예외를 추상화해 준다.

```java
@Bean
public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
  return new PersistenceExceptionTranslationPostProcessor();
}
```

* 예외를 그대로 반환하고 싶으면 `throws` 절에 그대로 반환할 JPA 예외나 JPA 예외의 부모 클래스를 직접 명시하면 된다.

```java
@Repository
public class NoResultExceptionService {
  
  @PersistenceContext 
  EntityManager em;
  
  public Member findMember() throws javax.persistence.NoResultException {
    return em.createQuery("selece m from Member m", Member.class).getSingleResult();
  }
}
```

##### 트랜잭션 롤백 시 주의사항

* 데이터베이스의 반영사항만 롤백하는 것이지 수정한 자바 객체까지 원상태로 복구하지는 않는다. 객체는 수정된 상태로 영속성 컨텍스트에 남아 있다.
* 새로운 영속성 컨텍스트를 사용하거나 영속성 컨텍스트를 초기화한 다음 사용하는 것이 안전하다.
* 트랜잭션당 영속성 컨텍스트 전략은 문제가 발생하면 트랜잭션 AOP 종료 시점에 트랜잭션을 롤백하면서 영속성 컨텍스트도 함께 종료한다.
* OSIV처럼 영속성 컨텍스트의 범위를 트랜잭션 범위보다 넓게 사용해서 여러 트랜잭션이 하나의 영속성 컨텍스트를 사용할 때 문제가 발생할 수 있다.
  * 스프렝은 트랜잭션 롤백 시 영속성 컨텍스트를 초기화해서 잘못된 영속성 컨텍스트를 사용하는 문제를 예방한다.