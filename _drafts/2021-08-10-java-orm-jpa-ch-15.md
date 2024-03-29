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



#### 엔티티 비교

* 영속성 컨텍스트 내부에는 엔티티 인스턴스를 보관하기 위한 1차 캐시가 있다. 영속성 컨텍스트와 생명 주기를 같이한다.
  * 영속성 컨텍스트를 통해 데이터를 저장하거나 조회하면 1차 캐시에 엔티티가 저장된다. 1차 캐시 덕에 변경 감지 기능도 동작하고, 이름 그대로 1차 캐시로 사용되어 데이터를 바로 조회할 수 있다.
* 1차 캐시의 가장 큰 장점은 애플리케이션 수준의 반복 가능한 읽기이다.
  * 같은 영속성 컨텍스트에서 엔티티를 조회하면 항상 같은 엔티티 인스턴스를 반환한다.
  * 단순히 동등성 비교가 아니라 정말 주소값이 같은 인스턴스를 반환한다.

##### 영속성 컨텍스트가 같을 때 엔티티 비교

![](https://user-images.githubusercontent.com/55083845/130626252-b9916485-c19c-4ed4-aa66-a3eafacc13eb.jpeg)

* 같은 트랜잭션 범위에 있으면 같은 영속성 컨텍스트를 사용한다. 따라서 영속성 컨텍스트가 같으면 엔티티를 비교할 때 아래 3가지 조건을 모두 만족한다.
  * 동일성: `==` 비교가 같다. 즉, 주소가 같다.
  * 동등성: `equals()` 비교가 같다. 
  * 데이터베이스 동등성: `@Id`인 데이터베이스 식별자가 같다.

##### 영속성 컨텍스트가 다를 때 엔티티 비교

![](https://user-images.githubusercontent.com/55083845/130627712-0e437c50-0f39-452b-b0ed-8b14724bd6ac.jpeg)

1. 테스트 코드에서 `memberService.join(member)`를 호출해서 회원가입을 시도하면 서비스 계층에서 트랜잭션이 시작되고 영속성 컨텍스트1이 만들어진다.
2. `memberRepository` 에서 `em.persist()`를 호출해서 `member` 엔티티를 영속화한다.
3. 서비스 계층이 끝날 때 트랜잭션이 커밋되면서 영속성 컨텍스트가 플러시된다. 이때 트랜잭션과 영속성 컨텍스트가 종료된다. 따라서 `member` 엔티티 인스턴스는 준영속 상태가 된다.
4. 테스트 코드에서 `memberRepository.findOne(saveId)` 를 호출해서 저장한 엔티티를 조회하면 레포지토리 계층에서 새로운 트랜잭션이 시작되며 새로운 영속성 컨텍스트2가 생성된다.
5. 저장된 회원을 조회하지만 새로 생성된 영속성 컨텍스트2에는 찾는 회원이 존재하지 않는다.
6. 따라서 데이터베이스에서 회원을 조회한다.
7. 조회된 엔티티를 영속성 컨텍스트에 보관하고 반환한다.
8. `memberRepository.findOne()` 메소드가 끝나며 트랜잭션이 종료되고 영속성 컨텍스트2도 종료된다.

* 영속성 컨텍스트1, 2에서 같은 아이디로 멤버를 찾았더라도 다른 영속성 컨텍스트에서 관리되었기 때문에 둘은 다른 인스턴스다.
  * 그러나 같은 데이터베이스 로우를 가르키고 있으므로 사실상 같은 엔티티이다.
* 즉, 영속성 컨텍스트가 다르면 동일성 비교에 실패한다. 영속성 컨텍스트가 다를 때 엔티티 비교는 아래와 같다.
  * 동일성: `==` 비교가 실패한다. 즉, 주소가 다르다.
  * 동등성: `equals()` 비교가 만족한다. 단, `equals`를 직접 구현해야 한다.
  * 데이터베이스 동등성: `@Id`인 데이터베이스 식별자가 같다.
* 같은 영속성 컨텍스트를 보장하면 동일성 비교만으로 충분하다.
* 영속성 컨텍스트가 달라지면 동일성 비교는 실패하므로 엔티티의 비교에 다른 방법을 사용해야 한다.
  * 데이터베이스 동등성 비교는 엔티티를 영속화해야 식별자를 얻을 수 있으므로 정확한 비교가 불가하다. 식별자 값을 직접 부여하는 방식인 경우 데이터베이스 식별자 비교도 가능하다. 그러나 항상 식별자를 먼저 부여하는 것을 보장하기는 어렵다.
  * 남는 것은 `equals` 를 사용한 동등성 비교인데, **엔티티 비교 시 비즈니스 키를 활용한 동등성 비교를 권장**한다.
* 동일성 비교는 같은 영속성 컨텍스트의 관리를 받는 영속 상태의 엔티티에만 적용 가능하다. 그렇지 않을 경우 비즈니스 키를 이용한 동등성 비교를 해야 한다.



#### 프록시 심화 주제

##### 영속성 컨텍스트와 프록시

