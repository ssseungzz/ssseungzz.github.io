---
title: "JPA 뽀개기 - 연관관계 매핑 기초"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 5. 연관관계 매핑 기초

객체의 참조와 테이블의 외래 키를 매핑하기

* 방향: 단방향과 양방향이 있다. 방향은 객체 관계에만 존재하고 테이블 관계는 항상 양방향이다.
* 다중성: 다대일, 일대다, 일대일, 다대다 다중성이 있다. 
* 연관관계의 주인: 객체를 양방향 연관관계로 만들면 연관관계의 주인을 정해야 한다.



#### 단방향 연관관계

* 객체 연관관계와 테이블 연관관계의 가장 큰 차이는 참조를 통한 연관관계(객체 연관관계)는 언제나 단방향이라는 점이다. 객체 간에 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다. 반면에 테이블은 외래 키 하나로 양방향으로 조인이 가능하다. 

![](data:image/png;base64,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)

* 객체는 참조(주소)로 연관관계를 맺는다. 테이블은 외래 키로 연관관계를 맺는다.
  * 연관된 데이터를 조회할 때 객체는 참조를 사용하지만 테이블은 조인을 사용한다.
  * 참조를 사용하는 객체의 연관관계는 단방향이고, 외래 키를 사용하는 테이블의 연관관계는 양방향(A JOIN B가 가능하면 B JOIN A도 가능)이다.
  * 객체를 양방향으로 참조하려면 단방향 연관관계를 두 개 만들어야 한다.

##### 순수한 객체 연관관계

* 객체는 참조를 사용해서 연관관계를 탐색할 수 있는데 이를 **객체 그래프 탐색**이라 한다.

##### 테이블 연관관계

* 데이터베이스는 외래 키를 사용해서 연관관계를 탐색할 수 있는데 이를 조인이라고 한다.

##### 객체 관계 매핑

* JPA를 사용해서 객체만 사용한 연관관계와 테이블만 사용한 연관관계를 매핑할 수 있다.

![](https://media.vlpt.us/post-images/conatuseus/7933e2a0-ded3-11e9-8f36-9fe4d9580a6d/image.png)

```java
@Entity
public class Member {
  
  @Id
  @Column(name = "MEMBER_ID")
  private String id;
  
  private String username;
  
  // 연관관계 매핑
  @ManyToOne
  @JoinColumn(name="TEAM_ID")
  private Team team;
  
  // 연관관계 설정
  public void setTeam(Team team) {
    this.team = team;
  }
  
  // Getter, Setter...
}
```

* 위의 예시에서 `Member.team` 과 `MEMBER.TEAM_ID`를 매핑하는 것이 연관관계 매핑이다. 연관관계 매핑을 위해 어노테이션이 사용된다.
  * `@ManyToOne`: 이름 그대로 다대일 관계를 나타낸다. 연관관계 매핑 시 다중성을 나타내는 어노테이션을 *필수로* 사용해야 한다.
  * `@JoinColumn(name="TEAM_ID")`:조인 컬럼은 외래 키를 매핑할 때 사용한다. `name` 속성에는 매핑할 외래 키 이름을 지정한다. 회원과 팀 테이블은 `TEAM_ID`로 연관관계를 맺으므로 이 값을 지정하면 된다. 이 어노테이션은 생략 가능하다.

##### @JoinColumn

* 외래 키를 매핑할 때 사용한다.
* `name`: 매핑할 외래 키 이름으로 기본 값은 필드명 + `_` + 참조하는 테이블의 기본 키 컬럼명
* `referencedColumnName`: 외래 키가 참조하는 대상 테이블의 컬럼명, 기본 값은 참조하는 테이블의 기본 키 컬럼명
* 그 외 나머지는 `@Column`과 동일

##### @ManyToOne

* 다대일 관계에서 사용한다.
* `optional`: `false`로 설정하면 연관된 엔티티가 항상 있어야 하며 기본 값은 `true`
* `fetch`: 글로벌 페지 전략 설정
* `cascade`: 영속성 전이 기능 설정
* `targetEntity`: 연관된 엔티티의 타입 정보를 설정, 거의 사용 X



#### 연관관계 사용

##### 저장

> JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다.

``` java
public void teamSave() {
  Team team1 = new Team("team1", "팀1");
  em.persist(team1);
  
  Member member1 = new Member("member1", "회원1");
  member1.setTeam(team1);
  em.persist(member1);
}
```

* 연관된 객체를 참조하고 저장한다. JPA는 참조한 식별자를 외래 키로 사용해서 적절한 등록 쿼리를 생성한다.

##### 조회

* 연관관계가 있는 엔티티를 조회하는 방법은 크게 2가지다.

  * 객체 그래프 탐색(객체 연관관계를 사용한 조회): 위의 예시에서는 `member1.getTeam()`을 통해 회원과 연관된 팀 엔티티를 조회할 수 있다. 이처럼 객체를 통해 연관된 엔티티를 조회하는 것을 객체 그래프 탐색이라 한다.

  * 객체 지향 쿼리 사용(*JPQL*):

    ```java
    private static void queryLogicJoin(EntityManager em) {
      String jpql = "select m from member m join m.team t where" + "t.name:=teamName";
      List<Member> resultList = em.createQuery(jpql, Member.class)
      		.setParameter("teamName", "팀1")
          .getResultList();
     ...
    }
    ```

    * 쿼리를 보면 회원이 팀과 관계를 가지고 있는 필드(`m.team`)를 통해서 회원과 팀을 조인했다.
    * `:`로 시작하는 것은 파라미터를 바인딩받는 문뻐이다.
    * JPQL은 객체(엔티티)를 대상으로 하고 SQL보다 간결하다.

##### 수정

* 수정은 따로 메소드가 존재하지 않으며 단순히 불러온 엔티티의 값만 변경해 두면 트랜잭션을 커밋할 때 플러시가 일어나면서 변경 감지 기능이 작동한다. 그리고 데이터베이스에 자동으로 반영한다.
* 이는 연관관계를 수정할 때도 같아서 참조하는 대상만 변경하면 나머지는 JPA가 자동으로 처리한다.

##### (연관관계) 삭제

* 연관관계를 `null`로 설정해 주면 된다.

##### 연관된 엔티티 삭제

* 연관된 엔티티를 삭제하려면 연관관계를 먼저 제거하고 삭제해야 한다. 그렇지 않으면 외래 키 제약 조건으로 인해 데이터베이스에서 오류가 발생한다. 팀1에 회원1과 회원2가 소속되어 있다면 팀1을 삭제하기 전에 연관관계를 먼저 제거해야 한다.



#### 양방향 연관관계

* 객체 연관관계는 단방향만 가능하므로 양방향 연관관계를 위해 반대쪽 엔티티에도 상대 객체의 참조를 설정해야 한다.

![](https://media.vlpt.us/post-images/conatuseus/3a0867c0-e529-11e9-9426-0f575ede7eb4/image.png)

* 데이터베이스 테이블은 외래 키 하나로 양방향으로 조회할 수 있으므로 추가할 내용이 없다.

##### 양방향 연관관계 매핑

```java
@Entity
public class Member {
  
  @Id
  @Column(name = "MEMBER_ID")
  private String id;
  
  private String username;
  
  // 연관관계 매핑
  @ManyToOne
  @JoinColumn(name="TEAM_ID")
  private Team team;
  
  // 연관관계 설정
  public void setTeam(Team team) {
    this.team = team;
  }
  
  // Getter, Setter...
}

/*---------------------------------------------*/

@Entity
public class Team {
  
  @Id
  @Column(name = "TEAM_ID")
  private String id;
  
  private String name;
  
  // 추가
  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<Member>();
  ...
}
```

* 팀과 회원은 일대다 관계이므로 팀 엔티티에 회원 컬렉션을 추가한다. 
* 일대다 관계를 매핑하기 위해 `@OneToMany`를 사용하는데, `mappedBy` 속성은 양방향 매핑일 때 사용되는 속성으로 반대쪽 매핑의 필드 이름을 값으로 주면 된다. (반대쪽 매핑이 `Member.team`이므로 `team`을 값으로 준다)
* 팀에서 회원 컬렉션으로 객체 그래프 탐색을 통해 회원을 조회할 수 있다.



#### 연관관계의 주인

* `mappedBy`는 왜 필요할까?
* 객체에는 양방향 연관관계가 없다. 서로 다른 단방향 연관관계 2개를 양방향인 것처럼 보이게 한다. 반면 데이터베이스 테이블은 **외래 키 하나로 두 테이블의 연관관계를 관리**한다.
* **엔티티를 단방향으로 매핑하면 참조를 하나만 사용**하므로 이 참조로 외래 키를 관리하면 된다. 그런데 엔티티를 양방향으로 매핑하면 A -> B, B -> A 두 곳에서 서로를 참조한다. 따라서 객체의 연관관계를 관리하는 포인트가 2곳이 된다.
  * 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나다. 따라서 둘 사이에 차이가 발생한다.
  * 이 차이로 인해 JPA에서는 **두 객체 연관관계 중 하나를 정해서 테이블의 외래 키를 관리**해야 하는데 이것을 **연관관계의 주인**이라 한다.

#####양방향 매핑의 규칙: 연관관계의 주인

* 양방향 연관관계 매핑 시 두 관계 중 하나를 연관관계의 주인으로 정해야 한다.
* **연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정, 삭제)할 수 있다.** 반면에 주인이 아닌 쪽은 *읽기만 가능*하다.
* 어떤 연관관계를 주인으로 정할지는 `mappedBy` 속성을 이용한다.
  * 주인은 `mappedBy` 속성을 사용하지 않는다.
  * 주인이 아니면 `mappedBy` 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야 한다.
* **연관관계의 주인을 정한다는 것은 외래 키 관리자를 선택**하는 것이다. 

##### 연관관계의 주인은 외래 키가 있는 곳

* 연관관계의 주인은 테이블에 외래 키가 있는 곳으로 정해야 한다. 주인이 아닌 곳에는 `mappedBy` 속성을 사용해서 주인이 아님을 설정한다. 속성의 값으로는 연관관계의 주인을 주면 된다.

![](https://media.vlpt.us/post-images/conatuseus/45ee5cf0-e5a3-11e9-8a66-09d286f71b9a/image.png)

* 연관관계의 주인만 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있다. 주인이 아닌 반대편은 읽기만 가능하고 외래 키를 변경하지는 못한다.

> 데이터베이스 테이블의 다대일, 일대다 관계는 항상 다 쪽이 외래 키를 가진다. 다 쪽인 `@ManyToOne`은 항상 연관관계의 주인이 되므로 `mappedBy` 속성이 없다.



#### 양방향 연관관계 저장

```java
public void testSave() {
  Team team1 = new Team("team1", "팀1");
  em.persist(team1);
  
  Member member1 = new Member("member1", "회원1");
  member1.setTeam(team1);
  em.persist(member1);
  
  Member member2 = new Member("member2", "회원2");
  member1.setTeam(team1);
  em.persist(member2);
}
```

* 연관관계의 주인(`Member.team`)인 필드를 통해서 연관관계를 설정하고 저장한다. 단방향 연관관계와 같은 방식이다.
* 양방향 연관관계는 연관관계의 주인이 외래 키를 관리한다. 따라서 주인이 아닌 방향은 값을 설정하지 않아도 데이터베이스에 외래 키 값이 정상 입력된다. 엔티티 매니저는 연관관계의 주인에 입력된 값을 사용해서 외래 키를 관리한다.



#### 양방향 연관관계의 주의점

* 연관관계의 주인에는 값을 입력하지 않고 연관관계의 주인이 아닌 곳에만 값을 입력하면 데이터베이스에 외래 키 값이 정상적으로 저장되지 않는다. **연관관계의 주인만이 외래 키의 값을 변경**할 수 있다.

##### 순순한 객체까지 고려한 양방향 연관관계

* 객체 관점에서 양쪽 방향에 모두 값을 입력해 주는 것이 가장 안전하다.
* 양쪽 모두 값을 입력하지 않으면 JPA를 사용하지 않는 순수한 객체 상태에서 문제가 발생할 수 있다.
  * JPA를 사용하지 않고 연관관계의 주인인 쪽에만 연관관계를 설정하면 연관관계의 주인이 아닌 쪽에서는 연관관계가 반영되지 않는다. 양방향은 양쪽 다 관계를 설정해야 한다.
  * JPA를 사용한 상태라면 양쪽에 연관관계를 설정해도 연관관계의 주인에 연관관계를 설정한 것만 저장 시에 사용되며, 반대쪽은 저장 시에 사용되지 않는다.

##### 연관관계 편의 메소드

* 양방향 연관관계에서는 연관관계 설정 메소드 안에서 양쪽 모두 연관관계를 설정하는 것이 좋다.
* 이렇게 한 번에 양방향 관계를 설정하는 메소드를 연관관계 편의 메소드라고 한다.

##### 연관관계 편의 메소드 작성 시 주의사항

* 연관관계를 변경할 때는 기존에 다른 연관관계가 존재하는지 확인하고 만약 존재한다면 해당 연관관계를 삭제하고 새로운 연관관계로 변경해 준다.
* 연관관계의 주인의 참조만 변경하면 문제없는 것처럼 보이지만 연관관계의 주인의 아닌 쪽을 변경하지 않고 영속성 컨텍스트가 살아 있다면 기존의 연관관계가 나타나게 된다.



#### 정리

* 단방향 매핑으로 테이블과 객체의 연관관계 매핑은 완료된다.
* 단방향을 양방향으로 만들면 반대 방향으로 객체 그래프 탐색 기능이 추가된다.
* 양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리해야 한다.
* 연관관계의 주인은 외래 키의 위치와 관련될 뿐 비즈니스 중요도와는 관련이 없다.

