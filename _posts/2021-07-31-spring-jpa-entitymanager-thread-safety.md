---
title: "Spring에서의 JPA EntityManager Thread-safety 보장"
categories:
  - Develop
tags:
  - jpa
  - springboot
toc: true
toc_label: "Contents"
toc_sticky: true
---

## Spring에서 JPA EntityManager의 Thread-safety 보장

* 스프링에서는 `@PersistenceContext` 어노테이션을 통해 엔티티 매니저를 주입받는다. 즉, 엔티티 매니저의 생성을 컨테이너에게 위임하고 필요할 때마다 어노테이션을 통해 의존성 주입으로 엔티티를 받아 사용하는 것이다.

```java
@Repository 
public class TestRepository {
  
  @PersistenceContext;
  private EntityManager em;
  
  public void save(SomeObject object) {
    em.persist(object);
  }
  
  // ...
}
```

>  그렇다면 엔티티 매니저는 어떻게 만들어질까?

* 엔티티 매니저는 엔티티 매니저 팩토리(`EntityManagerFactory`)에 의해 만들어진다.
* JPA는 `EntityManagerFactory`를 생성하는데, 이는 애플리케이션 로딩 시점에 *DB당 하나만* 생성된다.
* `@PersistenceContext` 내부를 보면 `EntityManagerFactoryUtils` 클래스에서 엔티티 매니저를 가져오는 것을 확인할 수 있다.

### 그래서 어떻게 Thread-safety를 보장한다는 거야?

* 엔티티 매니저의 thread-safety는 엔티티 매니저가 만들어지는 과정을 잘 보면 알 수 있는데, 엔티티 매니저를 프록시를 통해서 감싸고 엔티티 매니저의 메소드를 호출할 때마다 프록시를 통해서 엔티티 매니저를 생성한다.

```java
@Repository 
public class TestRepository {
  
  @PersistenceContext;
  private EntityManager em;
  
  public void save(SomeObject object) {
    System.out.println("em " + em.toString());
    em.persist(object);
  }
  
  // ...
}
```

```java
em = {$Proxy84@4415} "Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@4f644b12]"
```

* 정리하면 엔티티 매니저를 생성하는 엔티티 매니저 팩토리는 싱글톤으로 만들어지며, 엔티티 매니저는 생성될 때 프록시를 통해 감싸지고 엔티티 매니저의 메소드를 호출할 때마다 프록시를 통해서 엔티티 매니저를 생성함으로써 thread-safety할 수 있게 된다. 

> 참고; 트랜잭션과 thread-safety

* 엔티티 매니저는 실제 트랜잭션이 수행될 때 생성된다. 스레드 간 공유되면 안 되기 때문에 트랜잭션이 수행된 후에는 반드시 닫고 DB 커넥션을 반환한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FqwIFa%2FbtqHuiTZl68%2FKN047i4mpupk2Dt2XTcoXK%2Fimg.png)

* 엔티티 매니저가 생성되면 1:1로 영속성 컨텍스트가 생성되는데, 스프링 환경에서의 JPA에서는 여러 엔티티 매니저가 같은 트랜잭션 내라면 하나의 영속성 컨텍스트를 공유(트랜잭션 범위의 영속성 컨텍스트 전략)하게 된다. 또한 트랜잭션에 따라 접근하는 영속성 컨텍스트가 다르므로 같은 엔티티 매니저를 사용해도 멀티 스레드에 안전하다.

  