---
title: "JPA 뽀개기 - 트랜잭션과 락, 2차 캐시"
categories:
  - Develop
tags:
  - jpa
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 16. 트랜잭션과 락, 2차 캐시

#### 트랜잭션과 락

##### 트랜잭션과 격리 수준

* 트랜잭션은 ACID를 보장해야 한다.
  * 원자성(Atomicity): 트랜잭션 내에서 실행한 작업들은 마치 하나의 작업인 것처럼 모두 성공하든가 모두 실패해야 한다.
  * 일관성(Consistency): 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 한다. 예를 들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야 한다.
  * 격리성(Isolation): 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 않도록 격리한다. 예를 들어 동시에 같은 데이터를 수정하지 못하도록 해야 한다. 격리성은 동시성과 관련된 성능 이슈로 인해 격리 수준을 선택할 수 있다.
  * 지속성(Durability): 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다. 중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용해서 성공한 트랜잭션 내용을 복구해야 한다.
* 트랜잭션은 원자성, 일관성, 지속성을 보장한다. 문제는 격리성인데 완벽히 보장하려면 트랜잭션을 거의 차례대로 실행해야 한다.
  * 이는 동시성 처리 성능을 매우 나빠지게 한다.
* 이로 인해 트랜잭션 격리 수준을 4단계로 나누어 정의한다.
  * 아래 표의 위에서 아래로 갈수록 격리 수준이 높아진다. 격리 수준이 낮을수록 동시성은 증가하지만 격리 수준에 따른 문제가 발생한다.

| 격리 수준                            | DIRTY READ | NON-REPEATABLE READ | PHANTOM READ |
| ------------------------------------ | ---------- | ------------------- | ------------ |
| READ UNCOMMITTED(커밋되지 않은 읽기) | O          | O                   | O            |
| READ COMMITTED(커밋된 읽기)          |            | O                   | O            |
| REPEATABLE READ(반복된 읽기)         |            |                     | O            |
| SERIALIZABLE(직렬화 가능)            |            |                     |              |

* READ UNCOMMITTED: 커밋되지 않은 데이터를 읽을 수 있다. 예를 들어 트랜잭션1이 데이터를 수정하고 있는데 커밋하지 않아도 트랜잭션 2가 수정 중인 데이터를 조회할 수 있다. 이것을 DIRTY READ라 한다. 데이터 정합성에 심각한 문제가 발생할 수 있다.
* READ COMMITTED: 커밋한 데이터만 읽을 수 있다. 따라서 DIRTY READ가 발생하지 않는다. 하지만 NON-REPEATABLE READ는 발생할 수 있다. 예를 들어 트랜잭션 1이 회원 A를 조회 중인데 갑자기 트랜잭션 2가 회원 A를 수정하고 커밋하면 트랜잭션 1이 다시 회원 A를 조회했을 때 수정된 데이터가 조회된다. 이처럼 반복해서 같은 데이터를 읽을 수 없는 상태를 NON-REPEATABLE READ라 한다.
* REPEATABLE READ: 한 번 조회한 데이터를 반복해서 조회해도 같은 데이터가 조회된다. 하지만 PHANTOM READ는 발생할 수 있다. 예를 들어 트랜잭션 1이 10살 이하의 회원을 조회했는데 트랜잭션 2가 5살 회원을 추가하고 커밋하면 트랜잭션이 1이 다시 10살 이하의 회원을 조회했을 때 회원 하나가 추가된 상태로 조회된다. 이처럼 반복 조회 시 결과 집합이 달라지는 것을 PHANTOM-READ라 한다.
* SERIALIZABLE: 가장 엄격한 트랜잭션 격리 수준이다. PHANTOM-READ도 발생하지 않지만 동시성 처리 성능이 떨어질 수 있다.
* 애플리케이션 대부분은 동시성 처리가 중요하므로 데이터베이스들은 보통 READ COMMITTED 격리 수준을 기본으로 사용한다.
  * 데이터베이스마다 조금씩 다르다.

##### 낙관적 락과 비관적 락 기초

* JPA의 영속성 컨텍스트(1차 캐시)를 적절히 활용하면 데이터베이스 트랜잭션이 READ COMMITTED 수준이어도 애플리케이션 레벨에서 반복 가능한 읽기가 가능하다. 
  * 엔티티가 아닌 스칼라 값을 직접 조회하면 영속성 컨텍스트의 관리를 받지 못하므로 반복 가능한 읽기를 할 수 없다.
  * JPA는 데이터베이스 트랜잭션 격리 수준을 READ COMMITTED 정도로 가정한다. 일부 로직에 더 높은 격리 수준이 필요하면 낙관적 락이나 비관적 락 중 하나를 사용하면 된다.
* **낙관적 락**은 트랜잭션 대부분은 충돌이 발생하지 않는다고 가정한느 방법이다.
  * JPA가 제공하는 버전 관리 기능을 사용한다. 애플리케이션이 제공하는 락이다.
  * 트랜잭션을 커밋하기 전까지는 트랜잭션의 충돌을 알 수 없다는 특징이 있다.
* **비관적 락**은 트랜잭션의 충돌이 발생한다고 가정하고 락을 걸고 보는 방법이다.
  * 데이터베이스가 제공하는 락을 사용한다.
* 데이터베이스 트랜잭션 범위를 넘어서는 문제도 있다.
  * 예를 들어 사용자1과 사용자2가 동시에 제목이 같은 공지를 수정한다고 해 보자. 둘이 동시에 수정 화면을 열어 내용을 수정하는 중에 사용자1이 먼저 수정 완료 버튼을 눌렀다. 잠시 후에 2가 수정 완료 버튼을 눌렀다. 결과적으로 먼저 완료한 1의 수정 사항은 사라지고 나중에 완료한 2의 수정 사항만 남게 된다. 이를 **두 번의 갱실 분실 문제**라고 한다.
  * 트랜잭션의 범위를 넘어서므로 트랜잭션만으로는 문제를 해결할 수 없고, 세 가지 선택 방법이 있다.
    1. 마지막 커밋만 인정
    2. 최초 커밋만 인정
    3. 충돌하는 갱신 내용 병합
  * 기본은 마지막 커밋만 인정하기가 사용된다. 하지만 JPA가 제공하는 버전 관리 기능을 사용하면 최초 커밋만 인정하기를 구현할 수 있다. 충돌하는 갱신 내용 병합하기는 애플리케이션 개발자가 직접 사용자를 위해 병합 방법을 제공해야 한다.

##### @Version

* 낙관적 락을 사용하려면 `@Version` 어노테이션을 사용해서 버전 관리 기능을 추가해야 한다.
  * `Long(long)`, `Integer(int)`, `Short(short)`, `Timestamp`에 적용 가능하다.

```java
@Entity
public class Board {
  
  @Id
  private String id;
  private String title;
  
  @Version
  private Integer version;
}
```

* 버전 관리 기능을 적용하려면 엔티티에 버전 관리용 필드를 추가하고 `@Version` 을 붙인다. 엔티티를 수정할 때마다 버전이 하나씩 자동으로 증가한다.
  * 엔티티 수정 시 조회 시점의 버전과 수정 시점의 버전이 다르면 예외가 발생한다. 따라서 버전 정보를 사용하면 최초 커밋만 인정하기가 적용된다.

![](https://user-images.githubusercontent.com/55083845/129480819-44fc566e-602a-41be-9971-bae15655ee03.jpeg)

###### 버전 정보 비교 방법

* 엔티티를 수정하고 트랜잭션을 커밋하면 영속성 컨텍스트를 플러시하면서 UPDATE 쿼리를 실행한다. 이때 버전을 사용하면 엔티티면 검색 조건에 엔티티의 버전 정보를 추가한다.

```sql
UPDATE BOARD
SET
	TITLE=?,
	VERSION=? (버전 +1 증가)
WHERE
	ID=?
	AND VERSION=? (버전 비교)
```

* 데이터베이스 버전과 엔티티 버전이 같으면 데이터를 수정하면서 동시에 버전도 하나 증가시킨다. 만약 버전이 이미 증가해서 수정 중인 엔티티와 버전이 다르면 UPDATE 쿼리의 WHERE 문에서 VERSION 값이 다르므로 수정 대상이 없다. 이때는 버전이 이미 증가한 것으로 판단해 예외를 발생시킨다.
* 버전은 엔티티의 값을 변경하면 증가한다. 
  * 임베디드 타입과 갑 타입 컬렉션은 논리적인 개념상 해당 엔티티의 값이므로 수정하면 엔티티의 버전이 증가한다.
  * 연관관계 필드는 외래 키를 관리하는 연관관계의 주인 필드를 수정할 때만 버전이 증가한다.

##### JPA 락 사용

* 락은 다음 위치에 적용 가능하다.
  * `EntityManager.lock()`, `EntityManager.find()`, `EntityManager.refresh()`
  * `Query.setLockModel()`
  * `@NamedQuery`
* 조회하면서 즉시 락을 걸 수도 있고, 필요할 때 락을 걸 수도 있다.
* JPA가 제공하는 락 옵션은 `LockModeType` 에 정의되어 있다. 

| 락 모드   | 타입                        | 설명                                |
| --------- | --------------------------- | ----------------------------------- |
| 낙관적 락 | OPTIMISTIC                  | 낙관적 락을 사용                    |
| 낙관적 락 | OPTIMISTIC_FORCE_INCREMENT  | 낙관적 락 + 버전 정보를 강제로 증가 |
| 비관적 락 | PESSIMISTIC_READ            | 비관적 락, 읽기 락을 사용           |
| 비관적 락 | RESSIMISTIC_WRITE           | 비관적 락, 쓰기 락을 사용           |
| 비관적 락 | PESSIMISTIC_FORCE_INCREMENT | 비관적 락 + 버전 정보 강제 증가     |
| 기타      | NONE                        | 락 X                                |
| 기타      | READ                        | OPTIMISTIC과 같음                   |
| 기타      | WRITE                       | OPTIMISTIC_FORCE_INCREMENT와 같음   |

##### JPA 낙관적 락

* 낙관적 락은 버전을 사용한다. 따라서 낙관적 락을 사용하려면 버전이 있어야 한다. 낙관적 락은 트랜잭션을 커밋하는 시점에 충돌을 알 수 있다는 특징이 있다.
* 낙관적 락에서 발생하는 예외는 아래와 같다.
  * `OptimisticLockException`: JPA 예외
  * `StaleObjectStateException`: 하이버네이트 예외
  * `ObjectOptimisticLockingFailureException`: 스프링 예외 추상화
* 락 옵션 없이 `@Version` 만 있어도 낙관적 락이 적용된다. 옵션을 사용하면 더 세밀하게 제어할 수 있다.

###### NONE

* 엔티티에 `@Version`이 적용된 필트만 있으면 락 옵션 없이도 낙관적 락이 적용된다.
* 용도: 조회한 엔티티를 수정할 때 다른 트랜잭션에 의해 변경(삭제)되지 않아야 한다. 조회 시점부터 수정 시점까지를 보장한다.
* 동작: 엔티티를 수정할 때 버전을 체크하면서 버전을 증가한다. 이때 데이터베이스의 버전 값이 현재 버전이 아니면 문제가 발생한다.
* 이점: 두 번의 갱신 분실 문제를 예방한다.

###### OPTIMISTIC

* 엔티티를 조회만 해도 버전을 체크한다. 한 번 조회한 엔티티는 트랜잭션을 종료할 때까지 다른 트랜잭션에서 변경하지 않음을 보장한다.
* 용도: 조회 시점부터 트랜잭션이 끝날 때까지 조회한 엔티티가 변경되지 않음을 보장한다.
* 동작: 트랜잭션 커밋 시 버전 정보를 조회해 현재 엔티티의 버전과 같은지 검증한다. 아니라면 예외가 발생한다.
* 이점: DIRTY READ와 NON-REPEATABLE READ를 방지한다.

![](https://user-images.githubusercontent.com/55083845/129574480-ef74b163-6396-47a5-a33c-f405eb060e23.jpeg)

###### OPTIMISTIC_FORCE_INCREMENT

* 낙관적 락을 사용하면서 버전 정보를 강제로 증가한다.
* 용도: 논리적인 단위의 엔티티 묶음을 관리할 수 있다. 예를 들어 게시물과 첨부파일이 일대다, 다대일의 양방향 연관관계이고 첨부파일이 연관관계의 주인일 때 게시물을 수정하며 첨부파일만 추가한다고 해서 게시물의 버전은 증가하지 않는다. 그러나 해당 게시물은 논리적으로 변경되었다. 이때 게시물의 버전도 강제로 증가시키려면 OPTIMISTIC_FORCE_INCREMENT를 사용하면 된다.
* 동작: 엔티티를 수정하지 않아도 트랜잭션 커밋 시 갱신 쿼리를 사용해 버전 정보를 증가시킨다. 이때 데이터베이스의 버전이 엔티티의 버전과 다르면 예외가 발생한다. 추가로 엔티티를 수정하면 수정 시 버전 갱신이 발생한다.
* 이점: 강제로 버전을 증가해 논리적인 단위의 엔티티 묶음을 버전 관리할 수 있다.

![](https://user-images.githubusercontent.com/55083845/129575071-98302d25-4bf2-40d7-acd9-de40d5a41f4a.jpeg)

##### JPA 비관적 락

* 데이터베이스 트랜잭션 락 매커니즘에 의존하는 방법이다. 주로 SQL 쿼리에 `select for update` 구문을 사용하면서 시작하고 버전 정보는 사용하지 않는다.
* PESSIMISTIC_WRITE 모드를 사용한다.
* 엔티티가 아닌 스칼라 타입을 조회할 때도 사용할 수 있으며 데이터를 수정하는 즉시 트랜잭션 충돌을 감지할 수 있다.
* 비관적 락에서 발생하는 예외는 다음과 같다.
  * `PessimisticLockException` (JPA)
  * `PessimisticLockingFailureException` (스프링 예외 추상화)

###### PESSIMISTIC_WRITE

* 용도: 데이터베이스에 쓰기 락 걸기
* 동작: 데이터베이스 `select for update` 를 사용해서 락을 건다.
* 이점: NON-REPEATABLE READ를 방지한다. 락이 걸린 로우는 다른 트랜잭션이 수정할 수 없다.

###### PESSIMISTIC_READ

* 데이터를 반복 읽기만 하고 수정하지 않는 용도로 락을 걸 때 사용한다. 일반적으로 잘 사용하지 않는다.

###### PESSIMISTIC_FORCE_INCREMENT

* 비관적 락이지만 버전 정보를 강제로 증가시킨다. 하이버네이트는 `nowait` 를 지원하는 데이터베이스에 대해 `for update nowait` 옵션을 적용한다.
  * 오라클: `for update nowait`
  * PostgreSQL: `for update nowait`
  * `nowait` 지원하지 않으면 `for update`

##### 비관적 락과 타임아웃

* 비관적 락을 사용하면 락을 획득할 때까지 트랜잭션이 대기한다. 
* 타임아웃 시간을 줄 수 있다.

```java
Map<String, Object> properties = new HashMap<String, Object>();

properties.put("javax.persistence.lock.timeout", 10000);

Board board = em.find(Board.class, "boardId", LockModeType.PESSIMISTIC_WRITE, properties);
```



#### 2차 캐시

##### 1차 캐시와 2차 캐시

* 조회된 데이터를 메모리에 캐시해서 데이터베이스 접근 횟수를 줄이면 애플리케이션 성능을 개선할 수 있다.
  * 영속성 컨텍스트 내부에 엔티티를 보관하는 저장소가 있는데 이를 1차 캐시라고 한다.
  * 하이버네이트를 포함한 대부분의 JPA 구현체들은 1차 캐시 외에도 애플리케이션 범위의 캐시를 지원하는데 이것을 공유 캐시 또는 2차 캐시라 한다.

![](https://user-images.githubusercontent.com/55083845/129736027-c76ac00f-0240-42c1-931e-d2f0d0222d52.jpeg)

###### 1차 캐시

* 영속성 컨텍스트 내부에 있다. 엔티티 매니저로 조회하거나 변경하는 엔티티는 1차 캐시에 저장된다.
* 트랜잭션 커밋 혹은 플러시 호출 시 1차 캐시에 있는 엔티티의 변경 내역을 데이터베이스에 동기화한다.
* 컨테이너 위에서 실행하면 트랜잭션을 시작할 때 영속성 컨텍스트를 생성하고 트랜잭션을 종료할 때 영속성 컨텍스트도 종료한다. OSIV를 사용하면 요청의 시작부터 끝까지 같은 영속성 컨텍스트를 유지한다.
* 동작 방식과 특징은 아래와 같다.
  * 최초 조회 시 1차 캐시에 엔티티가 없으므로 데이터베이스에서 엔티티를 조회해 1차 캐시에 보관하고 1차 캐시에 보관한 결과를 반환한다.
  * 같은 엔티티가 있으면 해당 엔티티를 그대로 반환한다. 따라서 객체 동일성을 보장한다. 
* 기본적으로 영속성 컨텍스트 범위의 캐시다.

###### 2차 캐시

* 애플리케이션이 공유하는 캐시를 JPA는 공유 캐시라 하는데 일반적으로 2차 캐시라 부른다. 애플리케이션 범위의 캐시다.
* 2차 캐시를 적용하면 엔티티 매니저를 통해 데이터 조회 시 2차 캐시에서 찾고 없으면 데이터베이스에서 찾는다.
  1. 영속성 컨텍스트는 엔티티가 필요하면 2차 캐시를 조회한다.
  2. 2차 캐시에 엔티티가 없으면 데이터베이스를 조회해서 
  3. 결과를 2차 캐시에 보관한다.
  4. 2차 캐시는 자신이 보관하고 있는 엔티티를 복사해서 반환한다.
  5. 2차 캐시에 저장되어 있는 엔티티를 조회하면 복사본을 만들어 반환한다.
* 2차 캐시는 동시성을 극대화하려고 캐시한 객체를 직접 반환하지 않고 복사본을 만들어서 반환한다. 객체를 그대로 반환하면 여러 곳에서 객체를 동시에 수정하는 문제를 대비하기 위해 락을 걸어야 하는데 이렇게 하면 동시성이 떨어질 수 있기 때문이다.
* 2차 캐시의 특징은 아래와 같다.
  * 영속성 유닛 범위의 캐시다.
  * 조회한 객체를 그대로 반환하는 것이 아니라 복사본을 만들어서 반환한다.
  * 데이터베이스 기본 키를 기준으로 캐시하지만 영속성 컨텍스트가 다르면 객체 동일성을 보장하지 않는다.

##### JPA 2차 캐시 기능

###### 캐시 모드 설정

* 2차 캐시를 사용하려면 엔티티에 `@Cacheable` 어노테이션을 사용한다.

```java
@Cacheable
@Entity
public class Member {
  
  @Id
  @GeneratedValue
  private Long id;
  //...
}
```

* `persistence.xml`에 `shard-cache-mode`를 설정해서 애플리케이션 전체(정확히는 영속성 유닛 단위)에 캐시를 어떻게 적용할지 옵션을 설정해야 한다.
* 캐시 모드는 아래와 같다. 보통 ENABLE_SELECTIVE를 사용한다.

| 캐시 모드         | 설정                                                         |
| ----------------- | ------------------------------------------------------------ |
| ALL               | 모든 엔티티를 캐시한다.                                      |
| NONE              | 캐시를 사용하지 않는다.                                      |
| ENABLE_SELECTIVE  | Cacheable(true)로 설정된 엔티티만 캐시를 적용한다.           |
| DISABLE_SELECTIVE | 모든 엔티티를 캐시하는데 Cacheable(false)로 명시된 엔티티는 캐시하지 않는다. |
| UNSPCIFIED        | JPA 구현체가 정의한 설정을 따른다.                           |

###### 캐시 조회, 저장 방식 설정

* 캐시를 무시하고 데이터베이스를 직접 조회하거나 캐시를 갱신하려면 캐시 조회 모드와 캐시 보관 모드를 사용하면 된다.

```java
em.setProperty("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
```

* `javax.persistence.cache.retrieveMode`: 캐시 조회 모드 프로퍼티 이름
* `javax.persistence.cache.storeMode`: 캐시 보관 모드 프로퍼티 이름
* `javax.persistence.CacheRetrieveMode`: 캐시 조회 모드 설정 옵션, `USE`는 캐시에서 조회하는 기본 값이고 `BYPASS`는 캐시를 무시하고 데이터베이스에 직접 접근
* `javax.persistence.CacheStoreMode`: 캐시 보관 모드 설정 옵션, `USE`는 조회한 데이터를 캐시에 저장(갱신 X)하며 트랜잭션을 커밋하면 등록 수정한 엔티티도 캐시 저장하는 기본 값, `BYPASS`는 캐시에 저장하지 않고 `REFRESH`는 `USE`에 추가로 데이터베이스에 조회한 엔티티를 최신 상태로 다시 캐시
* 캐시 모드는 엔티티 매니저 단위로 설정하거나 메소드에 설정할 수도 있고, `Query.setHint`에 사용할 수 있다.

```java
Map<String, Object> param = new HashMap<String, Object>();
param.put("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS);
param.put("javax.persistence.cache.storeMode", CacheStoreMode.BYPASS);
em.find(TestEntity.class, id, param);

em.createQuery("select e from TestEntity e where id = :id",
              TestEntity.class)
              .setParameter("id", id)
  						.setHint("javax.persistence.cache.retrieveMode", CacheRetrieveMode.BYPASS)
  						.setHint("javax.persistence.cache.storeMode", CacheStoreMode.BYPASS)
  						.getSingleResult();
```

###### JPA 캐시 관리 API

* JPA는 `Cache` 인터페이스를 제공한다. `EntityManagerFactory`에서 구할 수 있다.
* 해당 엔티티가 캐시에 있는지 여부 확인, 해당 엔티티 중 특정 식별자를 가진 엔티티를 캐시에서 제거, 해당 엔티티 전체를 캐시에서 제거, 모든 캐시 데이터 제거, `JPA Cache` 구현체 조회 기능을 제공한다.

##### 하이버네이트와 ENCACHE 적용

* 하이버네이트가 지원하는 캐시는 크게 3가지가 있다.
  1. 엔티티 캐시: 엔티티 단위로 캐시한다. 식별자로 엔티티를 조회하거나 컬렉션이 아닌 연관된 엔티티를 로딩할 때 사용한다.
  2. 컬렉션 캐시: 엔티티와 연관된 컬렉션을 캐시한다. 컬렉션이 엔티티를 담고 있으면 식별자 값만 캐시한다. (하이버네이트 기능)
  3. 쿼리 캐시: 쿼리와 파라미터 정보를 키로 사용해서 캐시한다. 결과가 엔티티면 식별자 값만 캐시한다. (하이버네이트 기능)

###### 환경 설정

* `hibernate-ehcache` 라이브러리 추가
* `ehcache.xml`로 EHCACHE 설정
* `persistence.xml`에 캐시 정보를 추가해 하이버네이트에 캐시 사용 정보 설정
  * `hibernate.cache.use_second_level_cache`: 2차 캐시 활성화
  * `hibernate.cache.use_query_cache`: 쿼리 캐시 활성화
  * `hibernate.cache.region.factory_class`: 2차 캐시를 처리할 클래스 지정
  * `hibernate.generate_statistics`: 하이버네이트 통계 정보 출력

###### 엔티티 캐시와 컬렉션 캐시

```java
import javax.persistence.Cacheable;
import org.hibernate.annotations.Cache;

@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@Entity
public class ParentMember {
  
  @Id @GeneratedValue
  private Long id;
  
  private String name;
  
  @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
  @OneToMany(mappedBy = "parentMember", cascade = CascadeType.ALL)
  private List<ChildMember> childMembers = new ArrayList<ChildMember>();
  ...
}
```

* `@Cacheable`: 엔티티를 캐시하기 위한 어노테이션
* `@Cache`: 하이버네이트 전용으로 캐시와 관련된 더 세밀한 설정을 할 때 사용
* 위 예에서는 엔티티 캐시와 컬렉션 캐시(`ParentMember.childMembers`)가 적용된다.

###### @Cache

* 세밀한 캐시 설정이 가능하다.

| 속성    | 설명                                                         |
| ------- | ------------------------------------------------------------ |
| usage   | CacheConcurrencyStrategy를 사용해서 캐시 동시성 전략을 설정한다. |
| region  | 캐시 지역 설정                                               |
| include | 연관 객체를 캐시에 포함할지 선택한다. all, non-lazy 옵션을 선택할 수 있다. |

* 캐시 동시성 전략 속성

| 속성                 | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| NONE                 | 캐시를 설정하지 안흔다.                                      |
| READ_ONLY            | 읽기 전용으로 설정한다. 등록, 삭제는 가능하지만 수정은 불가하다. 읽기 전용인 불변 객체는 수정되지 않으므로 하이버네이트는 2차 캐시를 조회할 때 객체를 복사하지 않고 원본 객체를 반환한다. |
| NONSTRICT_READ_WRITE | 동시에 같은 엔티티를 수정하면 데이터 일관성이 깨질 수 있다. EHCACHE는 데이터를 수정하면 캐시 데이터를 무효화한다. |
| READ_WRITE           | 읽기 쓰기가 가능하고 READ COMMITTED 정도의 격리 수준을 보장한다. EHCACHE는 데이터를 수정하면 캐시 데이터도 같이 수정한다. |
| TRANSACTIONAL        | 컨테이너 관리 환경에서 사용할 수 있다. 설정에 따라 REPEATABLE READ 정도의 격리 수준을 보장받을 수 있다. |

* 캐시 종류에 따른 동시성 전략 지원 여부

| Cache             | read-only | nonstrict-read-write | read-write | transactional |
| ----------------- | --------- | -------------------- | ---------- | ------------- |
| ConCurrentHashMap | O         | O                    | O          |               |
| EHCache           | O         | O                    | O          | O             |
| Infinispan        | O         |                      | O          |               |

###### 캐시 영역

* 캐시를 적용한 코드느 캐시 영역에 저장된다.
* 엔티티 캐시 영역은 기본값으로 [패키지 명 + 클래스 명]을 사용하고 컬렉션 캐시 영역은 엔티티 캐시 영역 이름에 캐시한 컬렉션의 필드 명이 추가된다.
* 필요하면 `@Cache`의 `region` 속성으로 직접 지정할 수 있다.
* 캐시 영역을 위한 접두사를 사용하려면 `persistence.xml` 설정에 `hibernate.cache.region_prefix`를 사용하면 된다.
  * 캐시 영역이 정해져 있으므로 영역별로 세부 설정이 가능하다.

###### 쿼리 캐시

* 쿼리와 파라미터 정보를 키로 사용해서 쿼리 결과를 캐시한다.
* `hibernate.cache.use_query_cache` 옵션이 `true` 여야 한다. 
  * 쿼리 캐시를 적용하려는 쿼리마다 `org.hibernate.cacheable`을 `true`로 설정하는 힌트를 주변 된다.

```java
em.createQuery("select i from Item i", Item.class)
  	.setHint("org.hibernate.cacheable", true)
  	.getResultList();
```

###### 쿼리 캐시 영역

* 쿼리 캐시를 활서오하하면 다음 두 캐시 영역이 추가된다.

  * `org.hibernate.cache.internal.StandardQueryCache`: 쿼리 캐시를 저장하는 영역이다. 쿼리, 쿼리 결과 집합, 쿼리를 실행한 시점의 타임스탬프를 보관한다.
  * `org.hibernate.cache.spi.UpdateTimestampsCache`: 쿼리 캐시가 유효한지 확인하기 위해 쿼리 대상 테이블의 가장 최근 변경(등록, 수정, 삭제) 시간을 저장하는 영역이다. 테이블 명과 해당 테이블의 최근 변경된 타임스탬프를 보관한다.

* 쿼리 캐시는 캐시한 데이터 집합을 최신 데이터로 유지하기 위해 쿼리 캐시를 실행하는 시간과 쿼리 캐시가 사용하는 테이블들이 가장 최근에 변경된 시간을 비교한다.

  * 쿼리 캐시 적용 후 쿼리 캐시가 사용하는 테이블에 변경이 있으면 쿼리 결과를 다시 캐시한다.
  * 흐름은 다음과 같다.

  1. 쿼리를 실행하면 `StandardQueryCache` 영역에서 타임스탬프를 조회한다. 
  2. 쿼리가 사용하는 엔티티의 테이블을 `UpdateTimestampsCache` 캐시 영역에서 조회해서 테이블들의 타임 스탬프를 확인한다.
  3. 이때 `StandardQueryCache` 캐시 영역의 타임스탬프가 더 오래되었다면 캐시가 유효하지 않은 것으로 보고 다시 캐시한다.

* 쿼리 캐시를 잘 활용하면 성능 향상이 있지만 빈번하게 변경이 일어나는 테이블의 경우 성능이 저하될 수 있다.

> `UpdateTimestampsCache` 쿼리 캐시 영역은 만료되지 않도록 설정해야 한다. 해당 영역이 만료되면 모든 쿼리 캐시가 무효화된다. EHCACHE의 `eternal="true"` 옵션을 사용하면 캐시에서 삭제되지 않는다.

###### 쿼리 캐시와 컬렉션 캐시의 주의점

* 엔티티 캐시를 사용해서 엔티티를 캐시하면 엔티티 정보를 모두 캐시하지만 쿼리 캐시와 컬렉션 캐시는 결과 집합의 식별자 값만 캐시한다. 따라서 쿼리 캐시와 컬렉션 캐시를 조회하면 그 안에는 식별자 값만 있다. 이 값을 하나씩 엔티티 캐시에서 조회해서 실제 엔티티를 찾는다.
* 쿼리 캐시나 컬렉션 캐시만 사용하고 엔티티 캐시를 사용하지 않으면 성능 문제가 발생할 수 있다.
  1. `select m from Member m` 쿼리를 실행했는데 쿼리 캐시가 적용되어 있다. 결과 집합은 100건이다.
  2. 결과 집합에는 식별자만 있으므로 한 건씩 엔티티 캐시 영역에서 조회한다.
  3. `Member` 엔티티는 엔티티 캐시를 사용하지 않으므로 한 건씩 데이터베이스에서 조회한다.
  4. 100건의 SQL이 실행된다.
* 위와 같은 상황을 방지하려면 쿼리 캐시나 컬렉션 캐시를 사용할 때 대상 엔티티에도 꼭 엔티티 캐시를 적용해야 한다.
