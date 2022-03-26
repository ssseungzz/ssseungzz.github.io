---
title: "Modern Java in Action 17장; 리액티브 프로그래밍"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 17. 리액티브 프로그래밍
* 리액티브 프로그래밍에서는 다양한 시스템과 소스에서 들어오는 데이터 항목 스트림을 비동기적으로 처리하고 합쳐서 사용자에게 높은 응답성을 제공한다.
* 전체의 리액티브 시스템을 구성하는 여러 컴포넌트를 조절하는 데에도 리액티브 기법을 사용할 수 있으며 이런 방식으로 구성된 시스템에서는 고장, 정전 같은 상태에 대처할 뿐 아니라 다양한 네트워크 상태에서 메시지를 교환하고 전달할 수 있으며 무거운 작업을 하고 있는 상황에서도 가용성을 제공한다.

#### 17.1 리액티브 매니패스토
* 반응성: 리액티브 시스템은 빠를 뿐 아니라 더 중요한 특징으로 일정하고 예상할 수 있는 반응 시간을 제공한다. 결과적으로 사용자가 기대치를 가질 수 있다. 기대치를 통해 사용자의 확신이 증가하면서 사용할 수 있는 애플리케이션이라는 확인을 제공할 수 있다.
* 회복성: 장애가 발생해도 시스템은 반응해야 한다. 컴포넌트 실행 복제, 여러 컴포넌트의 시간(발송자와 수신자가 독립적인 생명 주기를 가짐)과 공간(발송자와 수신자가 다른 프로세스에서 실행됨) 분리, 각 컴포넌트가 비동기적으로 작업을 다른 컴포넌트에 위임하는 등 리액티브 매니패스토는 회복성을 달성할 수 있는 다양한 기법을 제시한다.
* 탄력성: 애플리케이션의 생명주기 동안 다양한 작업 부하를 받게 되는데 이 다양한 작업 부하로 애플리케이션의 반응성이 위협받을 수 있다. 리액티브 시스템에서는 작업 부하가 발생하면 자동으로 관련 컴포넌트에 할당된 자원 수를 늘린다.
* 메시지 주도: 회복셩과 탄력성을 지원하려면 약한 겨합, 고립, 위치 투명성 등을 지원할 수 있도록 시스템을 구성하는 컴포넌트의 경계를 명확하게 정의해야 한다. 비동기 메시지를 전달해 컴포넌트끼리의 통신이 이루어진다. 이 덕분에 회복성(장애를 메시지로 처리)과 탄력성(주고받은 메시지의 수를 감시하고 메시지의 양에 따라 적절하게 리소스를 할당)을 얻을 수 있다.

![](https://user-images.githubusercontent.com/55083845/159704840-5a645904-98a6-4b7c-88c4-68693f38ae81.jpeg)


##### 17.1.1 애플리케이션 수준의 리액티브
* 애플리케이션 수준 컴포넌트의 리액티브 프로그래밍의 주요 기능은 비동기로 작업을 수행할 수 있다는 점이다.
* 리액티브 프레임워크와 라이브러리는 스레드를 퓨처, 액터, 일련의 콜백을 발생시키는 이벤트 루프 등과 공유하고 처리할 이벤트를 변환하고 관리한다.
  * 스레드를 다시 쪼개는 종류의 기술을 이용할 때는 메인 이벤트 루프 안에서는 절대 동작을 블럭(데이터베이스나 파일 시스템 접근, 원격 서비스 호출 등)하지 않아야 한다는 전제 조건이 따른다.

##### 17.1.2 시스템 수준의 리액티브
* 리액티브 시스템은 여러 애플리케이션이 한 개의 일관적인, 회복할 수 있는 플랫폼을 구성할 수 있게 해 줄 뿐 아니라 이들 애플리케이션 중 하나가 실패해도 전체 시스템은 계속 운영될 수 있도록 도와주는 소프트웨어 아키텍처이다.
  * 애플리케이션을 조립하고 상호 소통을 조절한다. 리액티브 시스템의 주요 속성으로 메시지 주도를 꼽을 수 있다.
* 메시지는 정의된 목적지 하나를 향하는 반면, 이벤트는 관련 이벤트를 관찰하도록 등록한 컴포넌트가 수신한다는 점이 다르다. 리액티브 시스템에서는 수신자와 발신자가 각각 수신 메시지, 발신 메시지와 결합하지 않도록 이들 메시디를 비동기로 처리해야 한다.
  * 각 컴포넌트를 고립하려면 이들이 결합되지 않도록 해야 하며 그래야만 시스템이 장애(회복성)와 높은 부하(탄력성)에서도 반응성을 유지할 수 있다.
  * 컴포넌트에서 발생한 장애를 고립시킴으로 문제가 주변의 다른 컴포넌트로 전파되며 전체 장애로 이어지는 것을 막는다. 이로써 회복성을 제공할 수 있다. 고립과 비결합이 회복성의 핵심이다.
  * 탄력성은 위치 투명성, 즉 리액티브 시스템의 모든 컴포넌트가 수신자의 위치에 상관없이 다른 모든 서비스와 통신할 수 있음을 의미한다.

#### 17.2 리액티브 스트림과 플로 API
 * 리액티브 프로그래밍은 리액티브 스트림을 사용하는 프로그래밍이다. 리액티브 스트림은 잠재적으로 무한의 비동기 데이터를 순서대로 그리고 블록하지 않는 역압력을 전제해 처리하는 기술이다.
  * 부하가 발생한 컴포넌트는 이벤트 발생 속도를 늦추라고 알리거나 얼마나 많은 이벤트를 수신할 수 있는지 알리거나 다른 데이터를 받기 전에 기존의 데이터를 처리하는 데에 얼마나 시간이 걸리는지를 업스트림 발행자에게 알릴 수 있어야 한다.

##### 17.2.1 Flow 클래스 소개
* 리액티브 스트림 프로젝트의 표준에 따라 프로그래밍 발행-구독 모델을 지원할 수 있도록 `Flow` 클래스는 중첩된 인터페이스 네 개를 포함한다.
  * `Publisher`: 일련의 이벤트 제공, 역압력 기법에 의해 이벤트 제공 속도 제한됨, 구독자를 인수로 받는 `subscribe`라는 함수형 인터페이스로 구독자를 등록 가능
  * `Subscriber`
  * `Subscription`: 발행자와 구독자 사이의 제어 흐름, 역압력 관리
  * `Processor`
* 구독자 인터페이스는 발행자 인터페이스가 관련 이벤트를 발행할 때 호출할 수 있도록 콜백 메서드 네 개를 정의한다.
  * `onSubscribe`: 항상 처음 호출되고, 구독 개체를 전달
  * `onNext`: 여러 번 호출 가능
  * `onError`: 장애 발생 시 호출
  * `onComplete`: 마지막에 호출
* 구독 객체(`Subscription`)
  * `request`: 발행자에게 주어진 개수의 이벤트를 처리할 준비가 되었음을 알릴 수 있다.
  * `cancel`: 발행자에게 이벤트를 받지 않음을 통지한다.
* 플로 명세서에는 이들 인터페이스 구현이 어떻게 서로 협력해야 하는지 설명하는 규칙 집합을 정의한다.
  * 발행자는 반드시 구독의 `request` 메서드에 정의된 개수 이하의 요소만 구독자에게 전달해야 한다. 성공적으로 동작이 끝났으면 `onComplete`를 호출하고 문제가 발생하면 `onError`를 호출해 구독을 종료할 수 있다.
  * 구독자는 요소를 받아 처리할 수 있음을 발행자에게 알려야 한다. `onComplete`나 `onError` 신호를 처리하는 상황에서 구독자는 발행자나 객체의 어떤 메서드도 호출할 수 없으며 구독이 취소되었다고 가정해야 한다. 구독자는 `Subscription.request()` 메서드 호출이 없어도 언제든 종료 시그널을 받을 준비가 되어 있어야 하며 `Subscription.cancel()`이 호출된 이후에라도 한 개 이상의 `onNext`를 받을 준비가 되어 있어야 한다.
  * 발행자와 구독자는 정확하게 구독을 공유해야 하며 각각이 고유한 역할을 수행해야 한다. 그러려면 `onSubscribe`와 `onNext` 메서드에서 구독자는 `request` 메서드를 동기적으로 호출할 수 있어야 한다. `Subscription.cancel()` 메서드는 몇 번 호출해도 한 번 호출한 것과 같은 효과를 가져야 하며, 여러 번 호출해도 다른 추가 호출에 영향이 없도록 스레드에 안전해야 한다. 같은 구독자 객체에 다시 가입하는 것은 권장하지 않으나 이런 상황에서 예외가 발생해야 한다고 명세서가 강제하지는 않는다.
  ![](https://user-images.githubusercontent.com/55083845/160144956-234936e2-6618-433d-9735-66924a2e4054.jpeg)
  * `Processor` 인터페이스는 발행자와 구독자를 상속받을 뿐 아무 메서드도 추가하지 않는다. 
    * 리액티브 스트림에서 처리하는 이벤트의 변환 단계를 나타낸다.
    * 에러를 수신하면 이로부터 회속하거나 즉시 `onError` 신호로 모든 구독자에게 전파할 수 있다.
    * 마지막 구독자가 구독을 취소하면 자신의 업스트림 구독도 취소함으로 취소 신호를 전파해야 한다.
  * 자바 9 플로 API/리액티브 스트림 API는 구독자 인터페이스의 모든 메서드 구현이 발행자를 블록하지 않도록 강제하지만 이들 메서드가 이벤트를 동기적으로 처리해야 하는지, 비동기적으로 처리해야 하는지는 지정하지 않는다.

##### 17.2.2 첫 번째 리액티브 애플리케이션 만들기
```java

@Getter
public class TempInfo {

  public static final Random random = new Random();

  private final String town;
  private final int temp;

  public TempInfo(String town, int temp) {
    this.town = town;
    this.temp = temp;
  }

  public static Tempinfo fetch(String town) {
    if (random.nextInt(10) == 0) {
      throw new RuntimeException("Error!");
    }

    return new TempInfo(town, random.nextInt(100));
  }

  @Override
  public String toString() {
    return town + " : " + temp;
  }
}

```java
public class TempSubscription implements Subscription {

  private final Subscriber<? super TempInfo> subscriber;
  private String town;

  public TempSubscription(Subscriber<? super TempInfo> subscriber, String town) {
    this.subscriber = subscriber;
    this.town = town;
  }

  @Override
  public void request(long n) {
    for (long i = 0L; i < n; i++) {
      try {
        subscriber.onNext(TempInfo.fetch(town)); // 현재 온도를 구독자로 전달
      } catch (Exception e) {
        subscriber.onError(e);
        break;
      }
    }
  }

  @Override
  public void cancel() {
    subscriber.onComplete(); // 구독이 취소되면 완료 신호를 구독자로 전달
  }
}
```

```java
public class TempSubscriber implements Subscriber<TempInfo> {

  private Subscription subscription;

  @Override
  public void onSubscribe(Subscription subscription) {
    this.subscription = subscription;
    subscription.request(1); // 구독을 저장하고 첫 번째 요청을 전달
  }

  @Override
  public void onNext(TempInfo tempInfo) {
    System.out.println(tempInfo);
    subscription.request(1); // 다음 정보 요청
  }

  @Override
  public void onError(Throwable t) {
    System.out.println(t.getMessage());
  }

  @Override
  public void onComplete() {
    System.out.println("Done!");
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    getTemperatures("Seoul").subscribe(new TempSubscriber()); // 새 Publisher를 만들고 TempSubscriber를 구독시킴
  }

  private static Publisher<TempInfo> getTemperatures(String town) {
    return subscriber -> subscriber.onSubscribe(
      new TempSubscription(subscriber, town)); // 구독한 Subscriber에게 TempSubscription을 전송하는 Publisher 반환
    )
  }
}
```
* 위 코드에서는 구독자가 새로운 요소를 받을 때마다 구독 객체로 새 요청을 보내면 `request` 메서드가 구독자 자신에게 또 다른 요소를 보내는 문제가 있다. 이를 `Executor`를 추가한 다음 다른 스레드에서 구독자로 새 요소를 전달함으로써 해결할 수 있다.
```java
public class TempSubscription implements Subscription {
  //...
  private static final ExecutorService executor = Executors.newSingleThreadExecutor();

  @Override
  public void request(long n) {
    executor.submit(() -> {
      for (long i = 0L; i < n; i++> {
        try {
        subscriber.onNext(TempInfo.fetch(town)); // 현재 온도를 구독자로 전달
      } catch (Exception e) {
        subscriber.onError(e);
        break;
      }
      })
    })
  }
}
```

##### 17.2.3 Processor로 데이터 변환하기
* `Processor`는 구독자인 동시에 발행자다. 발행자를 구독한 다음 수신한 데이터를 가공해 다시 제공하는 데에 목적이 있다.
```java
public class TempProcessor implements Processor<TempInfo, TempInfo> {

  private Sbuscriber<? super TempInfo> subscriber;

  @Override
  public void subscribe(Subscriber<? super TempInfo> subscriber) {
    this.subscriber = subscriber;
  }

  @Override
  public void onNext(TempInfo temp) {
    subscriber.onNext(new TempInfo(temp.getTown(), (temp.getTemp - 32) * 5 / 9));
  }

  @Override
  public void onSubscribe(Subscription subscription) {
    subscriber.onSubscribe(subscription);
  }

  @Override
  public void onError(Throwable t) {
    subscriber.onError(t);
  }

  @Override
  public void onComplete() {
    subscriber.onComplete();
  }
}
```


```java
public class Main {
  public static void main(String[] args) {
    getTemperatures("Seoul").subscribe(new TempSubscriber()); // 새 Publisher를 만들고 TempSubscriber를 구독시킴
  }

  private static Publisher<TempInfo> getTemperatures(String town) {
    return subscriber -> {
      TempProcessor processor = new TempProcessor();
      processor.subscribe(subscriber);
      processor.onSubscribe(new TempSubscription(processor, town));
    }
    )
  }
}
```

##### 17.2.4 자바는 왜 플로 API 구현을 제공하지 않는가?
* API를 만들 당시 Akka, RxJava 등 다양한 리액티브 스트림의 자바 코드 라이브러리가 존재했기 때문이다.
  * 자바 9의 표준화 과정에서 이들 라이브러리는 공식적으로 `java.util.concurrent.FLow`의 인터페이스를 기반으로 리액티브 개념을 구현하도록 진화했다.

#### 17.3 리액티브 라이브러리 RxJava 사용하기
* RxJava는 자바로 리액티브 애플리케이션을 구현하는 데에 사용하는 라이브러리이다.
* RxJava는 리액티브 당김 기반 역압력 기능이 있는 `Flow`를 포함하는 `io.reactivex.Flowable` 클래스와 역압력을 지원하지 않는 `io.reactivex.Observable` 클래스를 지원한다.
  * RxJava는 이벤트 스트림을 두 가지 구현 클래스로 제공하면서 천 개 이하의 요소를 가진 스트림이나 마우스 움직임, 터치 이벤트 등 역압력을 적용하기 힘든 GUI 이벤트 그리고 자주 발생하지 않는 종류의 이벤트에 역압력을 적용하지 말 것을 권장한다.

##### 17.3.1 Observable 만들고 사용하기
* `Observable`(그리고 `Flowable`)은 `Publisher`를 구현하므로 팩토리 메서드는 리액티브 스트림을 만든다.
  * `just`는 한 개 이상의 요소를 이용해 이를 방출하는 `Observable`로 변환한다.
  * `interval`은 값을 방출할 시간 간격을 미리 정해 둘 수 있다.
* RxJava에서 `Observable`이 발행자, `Observer`가 구독자 역할을 한다.
  * `onSubscribe` 메서드는 `Subscription` 대신 `Disposable`을 인수로 갖는다는 점 자바 9 `Subscriber`와 같다.
  * `onNext` 메서드의 시그니처에 해당하는 람다 표현식을 전달해 발행자를 구독할 수 있다. 나머지는 기본 동작을 가지도록 할 수 있다.

```java
public static Observable<TempInfo> getTemparature(String town) {
  return Observable.create(emitter -> 
    Observable.interval(1, TimeUnit.SECONDS)
          .subscribe(i -> {
            if(!emitter.isDisposed()) { // 소비된 옵저버가 폐기되기 않았으면 작업 수행
              if (i >= 5) {
                emitter.onComplete();
              } else {
                try {
                  emitter.onNext(TempInfo.fetch(town));
                } catch (exception e) {
                  emitter.onError(e);
                }
              }
            }}));
}
```
* 필요한 이벤트를 전송하는 `ObservableEmitter`를 소비하는 함수로 `Observable`을 만들어 반환한다. RxJava의 `ObservableEmitter` 인터페이스는 RxJava의 기본 `Emitter`를 상속한다.
  * 새 `Disposable`을 설정하는 메서드와 시퀀스가 이미 다운스트림을 폐기했는지 확인하는 메서드 등을 제공한다.
```java
public interface Emitter<T> {
  void onNext(t);
  void onError(Throwable t);
  void onComplete();
}
```

```java
// 구독자

public class TempObserver implements Observer<TempInfo> {
  @Override
  public void onComplete() {
    System.out.println("done");
  }

  @Override
  public void onError(Throwable t) {
    System.out.println("got problem" + t.getMessage());
  }

  @Override
  public void onSubscribe(Disposable disposable) {

  }

  @Override
  public void onNext(TempInfo tempInfo) {
    System.out.println(tempInfo);
  }
}
```
* 역압력을 지원하지 않으므로 전달된 요소를 처리한 다음 추가 요소를 요청하는 `request()` 메서드가 필요하지 않다.

```java
public class Main {
  public static void main(String[] agrs) {
    Observable<TempInfo> observable = getTemperature("Seoul");
    observable.blockingSubscribe(new TempObserver());
  }
}
```

##### 17.3.2 Observable을 변환하고 합치기
* 한 스트림을 다른 스트림의 입력으로 사용하거나 관심 있는 요소만 거른 다른 스트림을 만들거나 매핑 함수로 요소를 변환하거나 두 스트림을 다양한 방법으로 합치는 등의 작업을 할 수 있다.
  * 이들 기능의 이해를 돕기 위해 리액티브 스트림 커뮤니티는 마블 다이어그램이라는 시각적 방법을 이용한다.

![](https://user-images.githubusercontent.com/55083845/160238657-83a929ba-3bd7-4fb8-8a98-c9584f08f895.jpeg)

```java
public static Observable<TempInfo> getCelsiusTemperature(String town) {
  return getTemperature(town)
            .map(temp -> new TempInfo(temp.getTown(),
                            (temp.getTemp() - 32) * 5 / 9);
}
```
* 메서드가 반환하는 `Observable`을 받아 매핑 함수가 적용된 요소를 방출하는 다른 `Observable`을 반환할 수 있다.
```java
public static Observable<TempInfo> getCelsiusTemparature(String... towns) {
  return Observable.merge(Arrays.stream(towns)
                          .map(TempObservable::getCelsiusTemperature)
                          .collect(toList()));
}
```
* `Observable`의 `Iterable`을 인수로 받아 마치 한 개의 `Observable`처럼 동작하도록 결과를 합칠 수 있다. 즉, 결과 `Observable`은 전달된 `Iterable`에 포함된 모든 `Observable`의 이벤트 발행물을 시간 순서대로 방출한다.