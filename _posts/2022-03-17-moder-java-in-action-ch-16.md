---
title: "Modern Java in Action 16장; CompletableFuture: 안정적 비동기 프로그래밍"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 16. CompletableFuture: 안정적 비동기 프로그래밍

#### 16.1 Future의 단순 활용
* `Future`를 이용하려면 시간이 오래 걸리는 작업을 `Callable` 객체 내부로 감싼 다음에 `ExecutorService`에 제출해야 한다.
* 자바 8 이전의 코드는 아래와 같다.
  * 오래 걸리는 작업이 끝나지 않으면 전체 작업이 끝나지 않는 문제가 있을 수 있으므로 스레드가 대기할 최대 타임아웃 시간을 설정하는 것이 좋다.
```java
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit(new Callable<Double>() {
  public Double call() {
    return doSomeLongComputation();
  }
});
doSomethingElse();
try {
  Double result = future.get(1, TimeUnit.SECONDS); // 비동기 작업의 결과를 가져온다. 결과가 준비되어 있지 않으면 호출 스레드가 블록되지만 최대 1초까지만 기다린다.
} catch (ExecutionException ee) {
  // 계산 중 예외 발생
} catch (InterruptedException ie) {
  // 현재 스레드에서 대기 중 인터럽트 발생
} catch (TimeoutException te) {
  // Future 완료 전 타임아웃 발생
}
```

##### 16.1.1 Future 제한
* `Future`는 비동기 계산이 끝났는지 확인할 수 있는 메서드, 계산이 끝나길 기다리는 메서드, 결과 회수 메서드 등을 제공하지만 아래와 같은 선언형 기능은 제공하지 않는다.
  * 두 개의 비동기 계산 결과를 하나로 합친다. 두 가지 계산 결과는 서로 독립적일 수 있으며 또는 두 번째 결과가 첫 번째 결과에 의존하는 상황일 수 있다.
  * `Future` 집합이 실행하는 모든 태스크의 완료를 기다린다.
  * `Future` 집합에서 가장 빨리 완료되는 태스크를 기다렸다가 결과를 얻는다.
  * 프로그램적으로 `Future`를 완료시킨다. (즉, 비동기 동작에 수동으로 결과 제공)
  * `Future` 완료 동작에 반응한다. (즉, 결과를 기다리면서 블록되지 않고 결과가 준비되었다는 알림을 받은 다음에 `Future`의 결과로 원하는 추가 동작을 수행할 수 있음)
* 이러한 기능을 선언형으로 이용할 수 있도록 자바 8에서는 `CompletableFuture` 클래스를 제공한다.
  * 람다 표현식과 파이프라이닝을 활용한다.

##### 16.1.2 CompletableFuture로 비동기 애플리케이션 만들기
* 전통적인 동기 API에서는 메서드를 호출한 다음 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 호출자는 반환된 값으로 다른 동작을 수행한다. 호출자와 피호출자가 다른 스레드에서 실행되는 상황이었더라도 피호출자의 동작 완료를 기다렸을 것이다. 이처럼 동기 API를 사용하는 상황을 블록 호출이라고 한다.
* 비동기 API에서는 메서드가 즉시 반환되며 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드게 할당한다. 이와 같은 비동기 API를 사용하는 상황을 비블록 호출이라고 한다.
  * 다른 스레드에 할당된 나머지 계산 결과는 콜백 메서드를 호출해서 전달하거나 호출자가 계산 결과가 끝날 때까지 기다린다는 메서드를 추가로 호출하면서 전달된다.

#### 16.2 비동기 API 구현(최저 가격 검색 애플리케이션)
```java
public class Shop {
  public double getPrice(String product) {
    // 데이터베이스 접근, 다른 외부 서비스 접근 등이 필요
  }
}
```
```java
public class Shop {
  public double getPrice(String product) {
    return calculatePrice(product);
  }

  private double calculatePrice(String product) {
    delay() // 1초 sleep - 동기
    return someCalculation();
  }
}

```

##### 16.2.1 동기 메서드를 비동기 메서드로 변환
```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new Completable<>();
  new Thread(() -> {
    double price = calculatePrice(product); // 다른 스레드에서 비동기적으로 계산 수행
    futurePrice.complete(price); // 오랜 시간이 걸리는 계산이 완료되면 Future에 값 설정
  }).start();
  return futurePrice; // 계산 결과가 완료되길 기다리지 않고 Future 반환
}
```

```java
Shop shop = new Shop("BestShop");
long start = System.nanoTime();
Future<Double> futurePrice = shop.getPriceAsync("my favorite product");
long invocationTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Invocation returned after " + invocationTime + "msecs");
doSomethingElse();
try {
  double price = futurePrice.get();
  System.out.println("Price is %.2f%n", price);
} catch (Exception e) {
  throw new RuntimeException(e);
}
long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
System.out.println("Price returned after " + retrieval + "msecs");
```
* 비동기 API가 즉시 `Future`를 반환하면 클라이언트는 그것을 이용해 나중에 결과를 얻을 수 있다.
  * 그 사이에 결과를 기다리면서 대기하지 않고 다른 작업을 처리할 수 있다.
  * 특별히 할 일이 없는 경우 `Future`의 `get` 호출해 결과값이 있다면 읽고, 없다면 값이 계산될 때까지 블록한다.


##### 16.2.2 에러 처리 방법
* 클라이언트는 타임아웃값을 받는 `get` 메서드의 오버로드 버전을 만들어 블록 문제를 해결할 수 있다. 그래야 다른 스레드에서 샐행되는 일에서 문제가 발생했을 때 클라이언트가 영원히 블록되지 않기 때문이다.
  * `CompletableFuture` 내부에서 발생한 예외를 알기 위해 `completeExceptionally` 메서드를 사용할 수 있다.
```java
public Future<Double> getPriceAsync(String product) {
  CompletableFuture<Double> futurePrice = new Completable<>();
  new Thread(() -> {
    try {
      double price = calculatePrice(product);
      futurePrice.complete(price);
    } catch (Exception ex) {
      futurePrice.completeExceptionally(ex); // 도중에 문제 발생 시 발생한 에러를 포함시켜 Future 종료
    }
  }).start();
  return futurePrice;
}
```

###### 팩토리 메서드 supplyAsync로 CompletableFuture 만들기
```java
public Future<Double> getPriceAsync(String product) {
  return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```
* `supplyAsync` 메서드는 `Supplier`를 인수로 받아 `CompletableFuture`를 반환한다.
  * `CompletableFuture`는 `Supplier`를 실행해서 비동기적으로 결과를 생성한다. 
  * `ForkJoinPool`의 `Executor` 중 하나가 `Supplier`를 실행할 것이다. 하지만 두 번째 인수를 받는 오버로드 버전의 메서드를 이용해 다른 `Executor`를 지정할 수 있다.

#### 16.3 비블록 코드 만들기
```java
public List<String> findPrices(String product) {
  return shops.stream()
        .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
        .collect(toList());
}
```
* 위 코드의 경우 동기로 순차적으로 정보를 요청하므로 오랜 시간이 걸린다.

##### 16.3.1 병렬 스트림으로 요청 병렬화하기
```java
public List<String> findPrices(String product) {
  return shops.parallelStream()
        .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
        .collect(toList());
}
```
* 병렬 스트림으로 병렬로 연산을 수행해 성능을 향상시킬 수 있다.

##### 16.3.2 CompletableFuture로 비동기 호출 구현하기 
```java
List<CompletableFuture<String>> priceFutures = 
        shops.stream()
        .map(shop -> CompletableFuture.supplyAsync(
          () -> String.format("%s price is %.2f",
           shop.getName(), shop.getPrice(product))))
        .collect(toList())
```
* `CompletableFuture`의 동작이 완료되고 결과를 추출한 다음에 리스트를 반환할 수 있도록 `map` 연산을 `List<CompletableFuture<String>>`에 적용할 수 있다.
  * 리스트의 모든  `CompletableFuture`에 `join`을 호출해 모든 동작이 끝나기를 기다린다. 이 메서드는 `Future` 메서드의 `get` 메서드와 같은 의미를 같지만 아무 예외도 발생시키지 않는다.
```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures =
          shops.stream()
          .map(shop -> CompletableFuture.suppleAsync(
            () -> shop.getName() + " price is " + 
                  shop.getPrice(product)))
          .collect(Collectors.toList());
  return priceFutures.stream()
          .map(CompletableFuture::join) // 모든 비동기 동작이 끝나길 기다린다.
          .collect(toList());
}
```
* 두 `map` 연산을 두 개의 스트림 파이프라인으로 처리했다.
  * 스트림 연산은 게으른 특성이 있으므로 하나의 파이프라인으로 연산을 처리하면 모든 가격 정보 요청 동작이 동기적, 순차적으로 이루어지는 결과가 된다. `CompletableFuture`로 요청 시 기존 요청 작업이 완료되어야 `join`이 결과를 반환하면서 다음 요소로 정보를 요청할 수 있기 때문이다.
* 그러나 여전히 병렬 스트림을 사용한 해결책보다 느리다.

##### 16.3.3 더 확장성이 좋은 방법
* `CompletableFuture`는 병렬 스트림에 비해 작업에 이용할 수 있는 다양한 `Executor`를 지정할 수 있다는 장점이 있다. 따라서 `Executor`로 스레드 풀의 크기를 조절하는 등 애플리케이션에 맞는 최적화된 설정을 만들어 성능을 향상시킬 수 있다.

##### 16.3.4 커스텀 Executor 사용하기
* 풀에서 관리하는 스레드 수를 어떻게 결정할 수 있을까?
  * 스레드 풀이 너무 크면 CPU와 메모리 자원을 서로 경쟁하느라 시간을 낭비할 수 있다. 반면 스레드 풀이 너무 작으면 CPU의 일부 코어는 활용되지 않을 수 있다. 다음 공식으로 대략적인 CPU 활용 비율을 계산할 수 있다.
  * N(threads) = N(CPU) * U(CPU) * (1 + W/C)
  * N(CPU)는 `Runtime.getRuntime().availableProcessors()`가 반환하는 코어 수
  * U(CPU)는 0과 1 사이의 값을 갖는 CPU 활용 비율
  * W/C는 대기시간과 계산시간의 비율
  * 스레드 수가 너무 많으면 오히려 서버가 크래시될 수 있으므로 하나의 `Executor`에서 사용할 스레드의 최대 개수는 100 이하로 설정하는 것이 바람직하다.
```java
private final Executor executor = 
        Executors.newFixedThreadPool(Math.min(shops.size(), 100), new ThreadFactory() {
          public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setDaemon(true); // 프로그램 종료를 방해하지 않는 데몬 스레드 사용
            return t;
          }
        });
```
```java
CompletableFuture.supplyAsync(() -> shop.getName() + " price is " +
                                    shop.getPrice(product), executor);
```
* 애플리케이션의 특성에 맞는 `Executor`를 만들어 `CompletableFuture`를 활용하는 것이 바람직하다.
* I/O가 포함되지 않은 계산 중심의 동작을 실행할 때는 스트림 인터페이스가 가장 구현하기 간단하며 효율적일 수 있다. (모든 스레드가 작업을 수행하는 상황에서는 프로세서 코어 수 이상의 스레드를 가질 필요가 없다.)
* 반면 작업이 I/O를 기다리는 작업을 병렬로 실행할 때는 `CompletableFuture`가 더 많은 유연성을 제공하며 대기/계산의 비율에 적합한 스레드 수를 설정할 수 있다. 특히 스트림의 게으른 특성 때문에 스트림에서 I/O를 실제로 언제 처리할지 예측하기 어려운 문제도 있다.

#### 16.4 비동기 작업 파이프라인 만들기
##### 16.4.1 할인 서비스 구현
```java
public class Quote {
  private final String shopName;
  private final double price;
  private final Discount.code discountCode;

  public Quote(String shopName, double price, Discount.Code code) {
    this.shopName = shopName;
    this.price = price;
    this.discountCode = code;
  }

  public static Quote parse(String s) {
    String[] split = s.split(":");
    String shopName = split[0];
    double price = Double.parseDouble(split[1]);
    Discount.Code discountCode = Discount.Code.valueOf(split[2]);
    return new Quote(shopName, price, discountCode);
  }

  // getter
}
```
```java
public class Discount {
  public enum Code {
    // 생략
  }

  public static String applyDiscount(Quote quote) {
    return quote.getShopName() + " price is " +
           Discount.apply(quote.getPrice(), quote.getDiscountCode());
  }

  private static double apply(double price, Code code) {
    delay(); // 응답 지연 흉내
    return format(price * (100 - code.percentage) / 100);
  }
}
```

##### 16.4.2 할인 서비스 사용
* 가장 간단하게(순차, 동기) 구현한 것은 아래와 같다.
```java
public List<String> findPrices(String product) {
  return shops.stream()
           .map(shop -> shop.getPrice(product))
           .map(Quote::parse)
           .map(Discount::applyDiscount)
           .collect(toList());
}
```

##### 16.4.3 동기 작업과 비동기 작업 조합하기
```java
public List<String> findPrices(String product) {
  List<CompletableFuture<String>> priceFutures = 
      shops.stream()
           .map(shop -> CompletableFuture.supplyAsync(
                                  () -> shop.getPrice(product), executor))
           .map(future -> future.thenApply(Quote::parse))
           .map(future -> future.thenCompose(quote ->
                                  CompletableFuture.supplyAsync(
                                    () -> Discount.applyDiscount(quote), executor)))
            .collect(toList());
  return priceFutures.stream()
              .map(CompletableFuture::join)
              .collect(toList())
}
```
* 첫 번째 연산에서는 비동기적으로 상점에서 정보를 조회한다.
* 두 번째 연산에서는 첫 번째 결과(`Stream<CompletableFuture<String>>`)를 이용해서 문자열을 `Quote`로 변환한다. 파싱 동작에는 원격 서비스나 I/O가 없으므로 즉시 동작을 수행할 수 있다. `thenApply` 메서드는 `CompletableFuture`가 끝날 때까지 블록하지 않는다. 즉, `CompletableFuture`가 동작을 완전히 완료한 다음에 `thenApply` 메서드로 전달된 람다 표현식을 적용할 수 있다. 따라서 `CompletableFuture<String>`을 `CompletableFuture<Quote>`로 변환한다.
* 세 번째 연산에서는 할인 전 가격에 원격 `Discount` 서비스에서 제공하는 할인율을 적용해야 한다. 원격 실행이 포함되므로 동기적으로 작업을 수행해야 한다.
  * 람다 표현식으로 이 동작을 팩토리 메서드(`supplyAsync`)에 전달하면 다른 `CompletableFuture`가 반환된다. 결국 여러 가지 `CompletableFuture`로 이루어진 연쇄적으로 수행되는 두 개의 비동기 동작을 만든 셈이 된다.
  * 두 비동기 연산을 파이프라인으로 만들 수 있도록 `thenCompose` 메서드를 제공한다. 이 메서드는 첫 번째 연산의 결과를 두 번째 연ㅅ난으로 전달한다. 즉, 첫 번째 `CompletableFuture`에 `thenCompose`를 호출하고 `Function`에 넘겨주는 식으로 두 `CompletableFuture`를 조합할 수 있다. `Function`은 첫 번째 `CompletableFuture` 반환 결과를 인수로 받고 두 번째 `CompletableFuture`를 반환하는데, 두 번째 것은 첫 번째의 결과를 계산의 입력으로 사용한다.
* 마지막으로  `CompletableFuture`가 완료되기를 기다렸다가 `join`으로 값을 추출할 수 있다.
 * `thenCompose` 메서드도 뒤에 `Async`로 끝나는 버전이 존재하는데, 그렇게 끝나지 않는 메서드는 이전 작업을 수행한 스레드와 같은 스레드에서 작업을 실행함을 의미하며 붙은 메서드는 다음 작업이 다른 스레드에서 샐행되도록 스레드 풀로 작업을 제출한다.

##### 16.4.4 독립 CompletableFuture와 비독립 CompletableFuture 합치기
* 독립적으로 실행된 두 개의 `CompletableFuture`의 결과를 합쳐야 하는 경우 `thenCombine` 메서드를 사용한다.
  * `BiFunction`을 두 번째 인수로 받아 결과를 어떻게 합칠지 정의하며, 마찬가지로 `Async`로 끝나는 버전이 존재한다.

##### 16.4.6 타임아웃 효과적으로 사용하기
* `orTimeout` 메서드는 지정된 시간이 지난 후에 `CompletableFuture`를 `TimeoutException`으로 완료하면서 또 다른 `CompletableFuture`를 반환할 수 있도록 내부적으로 `ScheduledThreadExecutor`를 활용한다.
```java
Future<Double> futurePriceInUSD = 
    CompletableFuture.supplyAsync(() -> shop.getPrice(product))
    .thenCombine(
      CompletableFuture.supplyAsync(
        () -> exchangeService.getRate(Money.EUR, Money.USD)),
        (price, rate) -> price * rate
      ))
    .orTimeout(3, TimeUnit.SECONDS);
```
* 미리 지정한 값을 사용하고자 하는 경우 `completeOnTimeout` 메서드를 이용한다.
```java
Future<Double> futurePriceInUSD = 
    CompleteFuture.supplyAsync(() -> shop.getPrice(product))
    .thenCombine(
      CompletableFuture.supplyAsync(
        () -> exchangeService.getRate(Money.EUR, Money.USD))
        .completeOnTimeout(DEFAULT_RATE, 1, TimeUnit.SECONDS),
        (price, rate) -> price * rate
      ))
    .orTimeout(3, TimeUnit.SECONDS);
```

#### 16.5 CompletableFuture의 종료에 대응하는 방법
* `get`이나 `join`으로 `CompletableFuture`가 완료될 때까지 블록하지 않고 결과가 제공될 때마다 즉시 사용할 수 있도록 한다.

##### 16.5.1 최저 가격 검색 애플리케이션 리팩터링
```java
public Stream<CompletableFuture<String>> findPricesStream(String product) {
  return shops.stream()
          .map(shop -> CompletableFuture.supplyAsync(
                                  () -> shop.getPrice(product), executor))
          .map(future -> future.thenApply(Quote::parse))
          .map(future -> future.thenCompose(quote ->
                   CompletableFuture.supplyAsync(
                     () -> Discount.applyDiscount(quote), executor)));
}
```
* 스트림에 네 번째 `map` 연산을 적용한다. 각 `CompletableFuture`에 동작을 등록한다. 등록된 동작은 계산이 끝나면 값을 소비한다.
  * `thenAccept`라는 메서드로 이 기능을 제공한다. 이 메서드는 연산 결과를 소비하는 `Consumer`를 인수로 받는다. 마찬가지로 `Async`가 붙은 버전도 제공된다.
  * 이 메서드는 `CompletableFuture`가 생성한 결과를 어떻게 소비할지 미리 지정했으므로 `CompletableFuture<Void>`를 반환한다.
```java
findPricesStream.("myPhone").map(f -> f.thenAccept(System.out::println));
```
* 가장 느린 결과도 소비하는 기회를 제공하기 위해서는 스트림의 모든 `CompletableFuture<Void>`를 배열로 추가하고 실행 결과를 기다려야 한다.
  * 팩토리 메서드 `allOf`는 `CompletableFuture` 배열을 입력으로 받아 `CompletableFuture<Void>`를 반환한다. 전달된 모든 `CompletableFuture`가 완료되어야 `Completable<Void>`가 완료된다. 따라서 이 메서드가 반환하는 `CompletableFuture`에 `join`을 호출하면 원래 스트림의 모든 `CompletableFuture`의 실행 완료를 기다릴 수 있다.
  * 반면 하나의 작업이 끝나길 기다리는 상황에서는 `anyOf`를 사용한다. `CompletableFuture` 배열을 입력으로 받아 `CompletableFuture<Object>`를 반환한다. 이는 처음으로 완료한 값으로 동작을 완료한다.
```java
CompletableFuture[] futures = findPricesStream("myPhone")
          .map(f -> f.thenAccept(System.out::println))
          .toArray(size -> new CompletalbeFuture[size]);
CompletableFuture.allOf(futures).join();
```