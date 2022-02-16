---
title: "Modern Java in Action 10장; 람다를 이용한 도메인 전용 언어"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 10. 람다를 이용한 도메인 전용 언어
* 도메인 전용 언어(DSL)로 애플리케이션의 비즈니스 로직을 표현하면 애플리케이션의 핵심 비즈니스를 모델링하는 소프트웨어 영역에서 읽기 쉽게 코드를 작성함으로써 버그를 방지할 수 있다.
* 스트림 API의 특성인 메서드 체인을 보통 자바의 루푸의 복잡함 제어와 비교해 유창함을 의미하는 플루언트 스타일이라고 부른다.
  * 이런 스타일은 쉽게 DSL에 적용할 수 있다.
  * 외부적 DSL은 외부에서 파싱 및 평가 API를 제공하는 것이 일반적이며, 내부적 DSL은 애플리케이션 수준의 기본값이 자바 메서드가 사용할 수 있도록 한 개 이상의 클래스 형식으로 노출된다.
* 기본적으로 DSL을 만들려면 애플리케이션 수준 프로그래머에 어떤 동작이 필요하며 이들을 어떻게 프로그래머에게 제공하는지 고민이 필요하다.
* 내부적 DSL에서는 유창하게 코드를 구현할 수 있도록 적절하게 클래스와 메서드를 노출하는 과정이 필요하다.

#### 10.1 도메인 전용 언어
* DSL은 특정 비즈니스 도메인의 문제를 해결하려고 만든 언어다.
  * 저수준 구현 세부 사항 클래스는 비공개로 만들어 사용자 친화적인 DSL을 만들 수 있다.
  * DSL을 개발할 때는 코드의 의도가 명확하게 전달되어 누구나 쉽게 이해할 수 있도록 해야 한다.

##### 10.1.1 DSL의 장점과 단점
* 장점
  * 간결함: API는 비즈니스 로직을 간편하게 캡슐화하므로 반복을 피할 수 있고 코드를 간결하게 만들 수 있다.
  * 가독성: 도메인 영역의 용어를 사용하므로 비 도메인 전문가도 코드를 쉽게 이해할 수 있다.
  * 높은 수준의 추상화: 도메인과 같은 추상화 수준에서 동작하므로 도메인의 문제와 직접적으로 연관되지 않은 세부 사항은 숨긴다.
  * 집중: 비즈니스 도메인의 규칙을 표현할 목적으로 설계된 언어이므로 프로그래머가 특정 코드에 집중할 수 있다.
  * 관심사 분리: 지정된 언어로 비즈니스 로직을 표현하므로 애플리케이션의 인프라 구조와 관련된 문제와 독립적으로 비즈니스 관련된 코드에서 집중하기 용이하다.
* 단점
  * 설계의 어려움: 간결하게 제한적인 언어에 도메인 지식을 담아야 한다.
  * 개발 비용: 초기 프로젝트에 많은 비용과 시간이 소모되며 유지 보수와 변경이 부담을 줄 수 있다.
  * 추가 우회 계층: DSL은 추가적인 계층으로 도메인 모델을 감싸며 계층을 최대한 작게 만들어 성능 문제를 회피한다.
  * 새로 배워야 하는 언어: 팀이 배워야 하는 언어가 늘어나게 된다.
  * 호스팅 언어 한계: 범용 프로그래밍 언어는 사용자 친화적 DSL을 만들기가 힘들다.

##### 10.1.2 JVM에서 이용할 수 있는 다른 DSL 해결책
###### 내부 DSL
* (사용 언어, 지금의 경우 자바) 자바로 구현한 DSL을 의미한다. 람다 표현식과 메서드 참조로 간단하고 표현력 있는 DSL을 만들 수 있게 되었다.
  * 기존 언어를 이용하므로 구현하는 데에 드는 노력이 줄어든다.
  * 나머지 코드와 함께 DSL을 컴파일할 수 있다.
  * 기존의 자바 IDE를 이용해 자동 완성, 자동 리팩터링 같은 기능을 사용할 수 있다.
###### 다중 DSL
* 같은 자바 바이트코드를 사용하는 JVM 기반 프로그래밍 언어를 이용함으로써 DSL 합침 문제를 해결할 수도 있다.
  * 새로운 언어를 배워야 한다.
  * 두 개 이상의 언어가 혼재하므로 여러 컴파일러로 소스를 빌드하도록 해야 한다.
  * 자바와 호환성이 완벽하지 않아 성능이 손실될 때가 있다.
###### 외부 DSL
* 자신만의 문법과 구문으로 새 언어를 설계해야 한다. 새 언어를 파싱하고, 파서의 결과를 분석하고, 외부 DSL을 실행할 코드를 만들어야 한다.
* 유연성이 무한하다.

#### 10.2 최신 자바 API의 작은 DSL
* 자바의 새로운 기능의 장점을 적용한 첫 API는 네이티브 자바 API 자신이다.
```java
Collections.sort(persons, new Comparator<Person>(){
  public int compare(Person p1, Person p2) {
    // ...
  }
})
Collections.sort(people, (p1, p2) -> p1.getAge() - p2.getAge()); // 내부 클래스를 람다로
Collections.sort(persons, comparing(p -> p.getAge())); // 정적 메서드 사용
Collections.sort(persons, comparing(Person::getAge));
```
* 위의 작은 API는 컬렉션 정렬 도메인의 최소 DSL이다. 

##### 10.2.1 스트림 API는 컬렉션을 조작하는 DSL
* `Stream` 인터페이스는 네이티브 자바 API에 작은 내부 DSL을 적용한 좋은 예다.
  * 컬렉션의 항목을 필더, 정렬, 변환, 그룹화, 조작하는 DSL로 볼 수 있다.
* 스트림 API의 플루언트 형식은 잘 설계된 DSL의 또 다른 특징이다. 모든 중간 연산은 게으르며 다른 연산으로 파이프라인될 수 있는 스트림으로 반환된다. 최종 연산은 적극적이며 전체 파이프라인이 계산을 일으킨다.

##### 10.2.2 데이터를 수집하는 DSL인 Collectors
* `Collector` 인터페이스를 이용해 스트림의 항목을 수집, 그룹화, 파티션하거나 필요한 `Collector` 객체를 만들고 합칠 수 있다.
```java 
Map<String, Map<Color, List<Car>>> carsByBrandAndColor =
    cars.stream().collect(groupingBy(Car::getBrand, groupingBy(Car::getColor))); // 다중 수준 그룹화
Comparator<Person> comparator = comparing(Person::getAge).thenComparing(Person::getName); // 플루언트 방식
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = groupingBy(Car::getBrand, groupingBy(Car::getColor)); // 다중 수준 Collector
```
* 셋 이상의 컴포넌트를 조합할 때는 플루언트 형식이 중첩 형식에 비해 가독성이 좋지만 중첩된 그룹화 수준에 반대로 그룹화 함수를 구현해야 한다. 여러 정적 메서드로 중첩한 경우 안쪽 그룹화가 처음 평가되고 코드에서는 가장 나중에 등장하게 된다.
```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector = groupingBy(Car::getBrand, groupingBy(Car::getColor))
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollectorFluent = groupOn(Car::getColor).after(Car::getBrand).get();
```
* 팩터리 메서드에 작업을 위임하는 빌더를 만들면 문제를 더 쉽게 해결할 수 있다. 빌더에 여러 그룹화 작업을 포함시킬 수 있기 때문이다.

#### 10.3 자바로 DSL을 만드는 패턴과 기법

##### 10.3.1 메서드 체인
* DSL에서 가장 흔한 방식 중 하나이다.
  * 플루언트 API로 몇 개의 빌더를 구현해야 한다.
  * 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요한 것이 단점이다.

##### 10.3.2 중첩된 함수 이용
* 다른 함수 안에 함수를 이용해 도메인 모델을 만든다.
  * 하나의 빌더에 여러 정적 메소드를 포함한다.
```java
Order order = order("BigBank",
                    buy(80, 
                          stock("IBM", on("NYSE")), at(125.00)),
                    sell(50,
                          stock("GOOGLE", on("NASDAQ")), at(375.00))
                  );
```
* 도메인 객체 계층 구조에 그대로 반영된다는 것이 장점이다.
  * 결과 DSL에 더 많은 괄호를 사용해야 하며, 인수 목록을 정적 메서드에 넘겨 주어야 한다.
  * 도메인 객체에 선택 사항 필드가 있으면 인수를 생략할 수 있으므로 여러 메서드 오버라이드를 구현해야 한다.
  * 인수의 의미가 이름이 아니라 위치에 의해 정의된다. 인수의 역할을 확실하게 만드는 여러 더미 메서드를 이용해 완화할 수 있다.

##### 10.3.3 람다 표현식을 이용한 함수 시퀀싱
```java
Order order = order(o -> {
    o.forCustomer("BigBank");
    o.buy(t -> {
      t.quantity(80);
      t.price(125.00);
      t.stock(s -> {
        s.symbol("IBM");
        s.market("NYSE");
      });
    });
    o.sell(t -> {
      t.quantity(50);
      t.price(375.00);
      t.stock(s -> {
        s.symbol("GOOGLE");
        s.market("NASDAQ");
      });
    });
})
```
* 람다 표현식을 받아 실행해 도메인 모델을 만들어 내는 여러 빌더를 구현한다.
  * 이들 빌더는 메서드 체인 패턴을 이용해 만들려는 객체의 중간 상태를 유지한다.
  * 메서드 체인 패턴에는 객체를 만드는 최상위 수준의 빌더를 가졌지만 이 경우에는 `Consumer` 객체를 빌더가 인수로 받음으로 DSL 사용자가 람다 표현식으로 인수를 구현할 수 있게 한다.

```java
public class LambdaOrderBuilder {
  private Order order = new Order();

  public static Order order(Consumer<LambdaOrderBuilder> consumer) {
    LambdaOrderBuilder builder = new LambdaOrderBuilder();
    consumer.accept(builder); // 주문 빌더로 전달된 람다 표현식 실행
    return builder.order; // OrderBuilder의 Consumer를 실행해 만들어진 주문 반환
  }

  public void forCustomer(String customer) {
    order.setCustomer(customer);
  }

  public void buy(Consumer<TradeBuilder> consumer) {
    trade(consumer, Trade.Type.BUY);
  }

  public void sell(Consumer<TradeBuilder> consumer) {
    trade(consumer, Trade.Type.SELL);
  }

  private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
    TradeBuilder builder = new TradeBuilder();
    builder.trade.setType(type);
    consumer.accept(builder); // TradeBuilder로 전달할 람다 표현식 실행
    order.addTrade(builder.trade); // TradeBuilder의 consumer를 실행해 만든 거래를 주문에 추가
  }
}
```
* 메서드 체인 패턴처럼 플루언트 방식으로 정의할 수 있고 중첩 함수 형식처럼 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지한다.
* 많은 설정 코드가 필요하며 자바 8 람다 표현식 문법에 의한 영향을 받는다.

##### 10.3.4 조합하기
* 여러 패턴을 혼용해 가독성 있는 DSL을 만들 수도 있다.
* 결과 DSL이 여러 기법을 혼용하고 있으므로 한 가지 기법을 적용한 것에 비해 DSL을 배우는 데에 오랜 시간이 걸린다.

##### 10.3.5 DSL에 메서드 참조 사용하기
```java
public class TaxCalculator {
  public DoubleUnaryOperator taxFunction = d -> d;

  public TaxCalculator with(DoubleUnaryOperator f) {
    taxFunction = taxFunction.andThen(f);
    return this;
  }

  public double calculate(Order order) {
    return taxFunction.applyAsDouble(order.getValue());
  }
}

double value = new TaxCalculator().with(Tax::regional)
                                  .with(Tax::surcharge)
                                  .calculate(order);
```
* 위의 방식을 통해 불리언 플래그를 인수로 받는 정적 메서드를 이용하거나 불리언 플래그를 설정하는 `TaxCalculator`를 이용하는 것보다 간결하고 유연하게 표현할 수 있다.

#### 10.4 실생활의 자바 8 DSL
* 메서드 체인
  * 메서드 이름이 키워드 인수 역할을 한다, 선택형 파라미터와 잘 동작한다, DSL 사용자가 정해진 순서로 메서드를 호출하도록 강제할 수 있다, 정적 메서드를 최소화하거나 없앨 수 있다, 문법적 잡음을 최소화할 수 있다는 장점이 있다.
  * 구현이 장황하며 빌드를 연결하는 접착 코드가 필요한 데다가 들여쓰기 규칙으로만 도메인 객체 계층을 정의한다는 단점이 있다.
* 중첩 함수
  * 구현의 장황함을 줄일 수 있으며 함수 중첩으로 도매엔 객체 계층을 반영한다.
  * 정적 메서드의 사용이 빈번하며 이름이 아닌 위치로 인수를 정의하는 데다가 선택형 파라미터를 처리한 메서드 오버로딩이 필요하다.
* 람다를 이용한 함수 시퀀싱
  * 선택형 파라미터와 잘 동작하며 정적 메서드를 최소화하거나 없앨 수 있고 람다 중첩으로 도메인 객체 계층 반영, 빌더의 접착 코드가 없다.
  * 구현이 장황하며 람다 표현식으로 인한 문법적 잡음이 존재한다.

##### 10.4.1 jOOQ
* SQL을 구현하는 내부적 DSL로 자바에 직접 내장된 형식 안전 언어다. 스트림 API와 조합해 사용할 수 있다.
```sql
SELECT * FROM BOOK
WHERE BOOK.PUBLISHED_IN = 2016
ORDER BY BOOK.TITLE
```
```java
create.selectFrom(BOOK)
      .where(BOOK.PUBLISHED_IN.eq(2016))
      .orderBy(BOOK.TITLE)
```

##### 10.4.2 큐컴버
* 개발자가 비즈니스 시나리오를 평문 영어로 구현할 수 있도록 도와주는 BDD 도구이다.
* 테스트 시나리오를 정의하는 스크립트는 제한된 수의 키워드를 제공하며 자유로운 형식으로 문장을 구현할 수 있는 외부 DSL을 활용한다.
  * 이들 문장은 테스트 케이스의 변수를 캡쳐하는 정규 표현식으로 매칭되며 테스트 자체를 구현하는 메서드로 이를 전달한다.
* 큐컴버 어노테이션을 이용해 테스트 시나리오를 구현할 수 있다.
  * 자바 8이 람다 표현식을 지원하면서 두 개의 인수 메서드(기존에 어노테이션 값을 포함한 정규 표현식과 테스트 메서드를 구현하는 람다)를 이용해 어노테이션을 제거하는 다른 문법을 큐컴버로 개발할 수 있다.

##### 10.4.3 스프링 통합
* 스프링 통합(Spring Integration)유명한 엔터프라이즈 통합 패턴을 지원할 수 있도록 의존성 주입에 기반한 스프링 프로그래밍 모델을 확장한다.
  * 복잡한 엔터프라이즈 통합 솔루션을 구현하는 단순한 모델을 제공하고 비동기, 메시지 주도 아키텍처를 쉽게 적용할 수 있게 돕는다.
  * 채널, 엔드포인트, 폴러, 채널 인터셉터 등 메시지 기반 어플리케이션에 필요한 가장 공통 패턴을 모두 구현한다.
```java
@Configuration
@EnableIntegration
public class MyConfiguration {

  @Bean
  public MessageSource<?> integerMessageSource() {
    MethodInvokingMessageSource source = 
            new MethodInvokingMesageSource();
    source.setObject(new AtomicInteger());
    source.setMethodName("getAndIncrement");
    return source;
  }

  @Bean
  public DirectChannel inputChannle() {
    return new DirectChannel();
  }

  @Bean
  public IntegrationFlow  myFlow() {
    return IntegrationFlows
            .from(this.integerMessageSource(),
              c -> c.poller(Pollers.fixedRate(10)))
            .channel(this.inputChannel())
            .filter((Integer p) -> p % 2 == 0)
            .transform(Object::toString)
            .channel(MessageChannels.queue("queueChannel"))
            .get(); 
  }
}
```