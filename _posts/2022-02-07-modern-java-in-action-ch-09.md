---
title: "Modern Java in Action 9장; 리팩터링, 테스팅, 디버깅"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 9. 리팩터링, 테스팅, 디버깅

#### 9.1 가독성과 유연성을 개선하는 리팩터링

##### 9.1.1 코드 가독성 개선
* 코드 가독성이 좋다는 것은 어떤 코드를 다른 사람도 쉽게 이해할 수 있음을 뜻한다.

##### 9.1.2 익명 클래스를 람다 표현식으로 리팩터링하기
```java
Runnable r1 = new Runnable() {
  public void run() {
    System.out.println("hello")
  }
};

Runnable r2 = () -> System.out.println("hello");
```
* 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.
  * 익명 클래스에서 `this`는 익명 클래스 자신을 가리키지만 람다에서 `this`는 람다를 감싸는 클래스를 가리킨다.
  * 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있지만 람다 표현식으로는 가릴 수 없다.
```java
int a = 10;
Runnable r1 = () -> {
  int a = 2; // 컴파일 에러
  System.out.println(a);
};
 Runnable r2 = new Runnable() {
   public void run() {
     int a = 2; // 잘 동작
     System.out.println(a);
   }
 }; 
```
* 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
  * 익명 클래스는 인스턴스화할 때 명시적으로 형식이 정해지는 반면 람다의 형식은 콘텍스트에 따라 달라진다.
  * 명시적 형변환을 이용해 모호함을 제거할 수 있다.
```java
interface Task() {
  public void execute();
}

public static void doSomething(Runnable r) { r.run(); }
public static void doSomething(Task t) { t.execute();

doSomething(new Task() {
  public void execute() {
    // ...
  }
});
doSomething(() -> System.out.println("hi")); // Runnable과 Task 중 무엇인지 모호함
doSomething((Task)() -> System.out.println("hi"));
```

##### 9.1.3 람다 표현식을 메서드 참조로 리팩터링하기
* 람다 표현식을 별도의 메서드로 추출한 다음 메서드 참조를 인수로 받는 메서드에 인수로 전달할 수 있다.
```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
    menu.stream().collect(groupingBy(Dish::getCaloricLevel));
```
* `comparing`, `maxBy` 같은 정적 헬퍼 메서드를 활용할 수 있다.
* `sum`, `maximum` 등 자주 사용하는 리듀싱 연ㅅ나은 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드를 제공한다.

##### 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기
* 스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여 주며 쇼트서킷과 게으름이라는 최적화뿐 아니라 멀티코어 아키텍처를 활용할 수 있게 해 준다.

##### 9.1.5 코드 유연성 개선
###### 함수형 인터페이스 적용
* 람다 표현식을 이용하려면 함수형 인터페이스가 필요하므로 함수형 인터페이스를 코드에 추가해야 한다.

###### 조건부 연기 실행
```java
if (logger.isLoggable(Log.FINER)) { // logger 상태 노출
  logger.finer("Problem" + generateDiagnostic());
} // 메시지 로깅할 때마다 logger 객체의 상태를 매번 확인
```
```java
logger.log(Level.FINER, "Problem" + generateDiagnostic()); // 인수로 전달된 메시지 수준에서 logger가 활성화되어 있지 않더라도 메시지 평가
```
* 람다를 이용하면 특정 조건에서만 실행되도록 연기(`defer`)할 수 있다.
```java
public void log(Level level, Supplier<String> msgSupplier) // 추가된 log 메서드의 시그니처
logger.log(Level.FINER, () -> "Problem" + generateDiagnostic());

public void log(Level level, Supplier<String> msgSupplier) { // 내부 구현
  if (logger.isLoggable(level)) {
    log("Problem" + generateDiagnostic());
  }
}
```

###### 실행 어라운드
* 매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드를 람다로 변환할 수 있다.
```java
String oneLine = processFile((BufferedReader b) - > b.readLine());
String twoLine = processFile((BufferedReader b) - > b.readLine() + b.readLine());

public static String processFile(BufferedReaderProcessor p) throws IOException {
  try (BufferedReader br = new BufferedReader(new
       FileReader("fileLocation"))) {
         return p.process(br);
       }
}

public interface BufferedReaderProcessor {
  String process(BufferedReader b) throws IOException;
}
```

#### 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

##### 9.2.1 전략
* 전략 패턴은 세 부분으로 구성된다.
  * 알고리즘을 나타내는 인터페이스(`Strategy` 인터페이스)
  * 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
  * 전략 객체를 사용하면 한 개 이상의 클라이언트
* 함수형 인터페이스를 사용하므로 다양한 전략을 구현하는 새로운 클래스를 구현할 필요 없이 람다 표현식을 직접 전달할 수 있다.
```java
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
```

##### 9.2.2 템플릿 메서드
* 알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 사용한다.
* 알고리즘의 개요를 만든 후 구현자가 원하는 기능을 추가할 수 있게 람다를 사용할 수 있다.
```java
abstract class OnlineBanking {
  public void processCustomer(int id) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy(c);
  }
  abstract void makeCustomerHappy(Customer c);
  public void processCustomer(int it, Consumer<Customer> makeCustomerHappy) {
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
  }
}

new OnlineBankingLambda().processCustomer(1, (Customer c) -> System.out.println("hi" + c.getName()));
```

##### 9.2.3 옵저버
* 어떤 이벤트가 발생했을 때 한 객체(주제라 불리는)가 다른 객체 리스트(옵저버라 불리는)에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.
* 새 옵저버를 명시적으로 인스턴스화하지 않고 람다 표현식을 직접 전달해서 실행할 동작을 지정할 수 있다.
```java
subject.registerObserver((String post) -> {
  if (post != null && post.contains("money")) {
    System.out.println("news" + post);
  }
})
```

##### 9.2.4 의무 체인
* 작업 처리 객체의 체인(동작 체인 등)을 만들 때는 의무 체인 패턴을 사용한다.
* 일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다.
* 작업 처리 객체를 `UnaryOperator<T>` 형식의 인스턴스로 표현하고 `andthen` 메서드로 이들 함수를 조합해서 체인을 만들 수 있다.
```java
UnaryOperator<String> headerProcessing = (String text) -> "From a, b" + text;
UnaryOperator<String> spellCheckProcessing = (String text) -> text.replaceAll("a", "c");
Function<String, String> pipeLine = headerProcessing.andThen(spellCheckProcessing);
String result = pipeline.apply("semple text");
```

##### 9.2.5 팩토리
* 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.
* 생성자도 메서드 참조처럼 접근할 수 있다.
```java
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
  map.put("loan", Loan::new);
  map.put("stock", Stock::new);
  map.put("bond", Bond::new);
}
public static product createProduct(String name) {
  Supplier<Product> p = map.get(name);
  if (p != null) return p.get();
  throw new IllegalArgumentException("no such product " + name);
}
```
* 여러 인수가 필요한 경우 시그니처가 복잡해질 수 있다.

#### 9.3 람다 테스팅

##### 9.3.1 보이는 람다 표현식의 동작 테스팅
* 필요하다면 람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.
* 람다 표현식은 함수형 인터페이스의 인스턴스를 생성하므로 생성된 인스턴스의 동작으로 람다 표현식을 테스트할 수 있다.

##### 9.3.2 람다를 사용하는 메서드의 동작에 집중하라
* 람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것이고, 그러려면 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다.
* 람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.

##### 9.3.3 복잡한 람다를 개별 메서드로 분할하기
* 람다 표현식을 메서드 참조로 바꿔서(새로운 일반 메서드 선언) 일반 메서드를 테스트하듯이 람다 표현식을 테스트할 수 있다.

##### 9.3.4 고차원 함수 테스팅
* 함수를 인수로 받거나 다른 함수를 반환하는 메서드(고차원 함수)는 좀 더 사용하기 어렵다.
* 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트할 수 있다.
  * 프레디케이트로 `filter` 메서드를 테스트할 수 있다.
```java
@Test
void testFilter() throws Exception {
  List<Integer> nums = Arrays.asList(1, 2, 3, 4);
  List<Integer> even = filter(nums, i -> i % 2 == 0);
  List<Integer> smallerThanThree = filter(nums, i -> i < 3);
  assertEquals(Arrays.asList(2, 4), even);
  assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```
* 테스트해야 할 메서드가 다른 함수를 반환하면 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트할 수 있다.

#### 9.4 디버깅
##### 9.4.1 스택 트레이스 확인
* 람다 표현식은 이름이 없어 조금 복잡한 스택 트레이스가 생성된다.
* 메서드 참조를 사용하는 클래스와 같은 곳에 선언되어 있는 메서드를 참조할 때는 메서드 참조 이름이 스택 트레이스에 나타난다.

###### 9.4.2 정보 로깅
* `peek`이라는 스트림 연산은 스트림의 각 요소를 소비한 것처럼 동작을 실행하지만 실제로 요소를 소비하지는 않으면서 자신이 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달한다.
```java
numbers.stream()
    .peek(x -> System.out.println("from stream: " + x))
    .map(x -> x + 17)
    .peek(x -> System.out.println("from stream: " + x))
    .filter(x -> x % 2 == 0)
    .peek(x -> System.out.println("from stream: " + x))
    .collect(toList())
```