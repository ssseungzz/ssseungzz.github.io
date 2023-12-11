### OptimisticLockingFailureException 이슈 탐색기

> 딱히 해결은 하지 않았지만 아무튼 이슈가 있었고, 그걸 위해 여러가지로 찾아본 경험담이다.

#### Optimistic Lock이란?

* `낙관적 잠금 실패` 예외를 해결하려면 우선 낙관적 잠금(Opmistic Lock)이 뭔지 알아야 할 것이다.
* 낙관적 잠금이란 **데이터 갱신** 시 **충돌이 발생하지 않을 것**이라고 보고 낙관적으로 잠금을 거는 기법이다.

> 낙관적으로 잠금을 건다는 게 뭘까?

* 낙관적이라는 말은 자원 경쟁을 낙관적으로 본다는 의미이다. 즉, 다중 트랜잭션이 데이터를 동시에 수정하지 않을 거라고 본다는 것이다.
* 따라서 데이터를 읽을 때 락을 설정하지 않는다.
  * 하지만 이는 당연하게도 데이터 갱신 시점에서 잘못된 데이터 갱신을 방지하지 못한다.
* 그래서 데이터 갱신 시점에 앞에서 읽은 데이터가 다른 사용자에 의해 변경되었는지 검사해야 한다.

#### JPA에서 낙관적 잠금의 사용

* 데이터 읽기에 락을 설정하지 않는 것까지는 그렇다 치고, JPA에서는 어떻게 데이터 갱신 시점에 앞에서 읽은 데이터가 다른 사용자(혹은 스레드)에 의해 변경되었는지 검사할 수 있을까?
* 바로 엔티티 내부에 `@Version` 을 사용함으로써 가능해진다. 데이터가 갱신될 때마다 버전이 증가하고, 데이터 조회 시의 버전과 데이터 갱신 시점의 버전이 같은지를 확인하는 것이다.
  * `@Version`에는 몇 가지 제약이 있다.
  * 하나의 엔티티는 하나의 필드에만 버전 annotation을 사용할 수 있고, 여러 테이블에 매핑된 엔티티의 경우 기본 테이블에 배치되어야 한다. 버전에 명시된 타입은 `int(Integer)`, `long(Long)`, `short(Short)`, `java.sql.TimeStamp` 중 하나여야 한다.

```java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String author;
    private Double price;
    
    @Version
    private Long version;

    // getters and setters

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public Long getVersion() {
        return version;
    }

    public void setVersion(Long version) {
        this.version = version;
    }
}
```

* 만약 데이터 조회 시의 버전과 데이터 갱신 시점의 버전이 다르면 `OptimisticLockingFailureException`이 발생하게 된다. 
* 이 예외가 발생하면 트랜잭션은 롤백 처리된다. 일반적으로는 엔티티를 다시 로드해서 업데이트를 다시 시도한다. 대략 아래와 같은 플로우로 진행된다.

```
         Client 1                          Server                             Client 2
            |                                  |                                  |
            |  Request to update resource      |                                  |
            |--------------------------------->|                                  |
            |                                  |  Respond with current version     |
            |                                  |  of resource                     |
            |<---------------------------------|                                  |
            |  Modify resource and set version |                                  |
            |  number to current version + 1   |                                  |
            |                                  |                                  |
            |  Request to update resource with |                                  |
            |  changes and new version number  |                                  |
            |--------------------------------->|                                  |
            |                                  |  Check if version number matches |
            |                                  |  current version on server       |
            |                                  |                                  |
            |                                  |  Version numbers don't match     |
            |                                  |  respond with conflict message   |
            |                                  |--------------------------------->|
            |  Response with conflict message  |                                  |
            |<---------------------------------|                                  |
            |                                  |                                  |
            |  Request to get updated version  |                                  |
            |--------------------------------->|                                  |
            |                                  |  Respond with updated version    |
            |                                  |  of resource                     |
            |<---------------------------------|                                  |

```



#### 낙관적 잠금 예외 발생 시의 해결 - 재시도

* 위에 적었듯 예외가 발생하면 일반적으로는 엔티티를 다시 로드해서 업데이트를 다시 시도한다. 나는 처음에 재시도를 편하게 하려고 `@Retryable` 을 사용했는데, `@Transactional`이 달린 메서드에 `@Retryable`을 사용해서인지 원하는 대로 재시도가 되지 않았다.
  * 두 어노테이션 모두 Spring AOP를 사용하는데, 
* 그래서 다른 걸 사용했고 어쩌고...
* 우리 팀에서의 이슈의 경우 테스트 망에서 동일한 내용의 요청이 단 몇 ns 차이로 와서 발생한 거였는데, 보내는 쪽의 버그이기도 하고 요청 데이터 자체는 신뢰할 수 있기 때문에 첫 요청 업데이트 이후 발생하는 낙관적 잠금 에러를 위해 굳이 코드를 수정하지는 않는 방향으로 결정되었다는 했다는 후문(...).  