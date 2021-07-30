---
title: "[Design Pattern] Observer Pattern 알아보기"
categories:
  - Develop
tags:
  - "design-pattern"
  - observer
toc: true
toc_label: "Contents"
toc_sticky: true
---

## 옵저버 패턴에 대해 알아보기

### 옵저버 패턴?

![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8d/Observer.svg/854px-Observer.svg.png)

> 다이어그램으로 모든 설명을 끝낼 수 있을 만큼 단순하다.

* 객체들 사이에 일대다 관계를 정의한다. 
* 주제 혹은 `Observable` 객체는 동일한 인터페이스를 써서 옵저버에 연락한다. 연락을 받는 옵저버는 전달된 데이터로 자신이 필요한 연산을 수행한다. 
  * `Observable`에서는 옵저버들이 `Observer` 인터페이스를 구현한다는 것을 제외하고 옵저버에 대해 아는 것이 없다. 따라서 느슨한 결합이 가능하다.
  * 신문 구독(출판사 - 독자)과 비슷한 매커니즘으로 생각할 수 있다.
* 옵저버 패턴을 이용하면 주제 객체에서 데이터를 보내거나(push) 옵저버가 데이터를 가져오는 방식(pull)을 사용할 수 있다. 
* 옵저버들한테 연락을 돌리는 순서에 의존해서는 안 된다. 
* 패턴의 구성 요소들의 의미는 아래와 같다.
  * `Subject`: 주제를 나타낸다. 옵저버로 등록하거나 옵저버 목록에서 탈퇴할 수 있는 메소드를 가지고 있다. 옵저버들에게 연락(정보 전달)하는 것을 담당한다. - 보통 옵저버와 마찬가지로 인터페이스와 구상 클래스를 분리한다.
  * `Observer`: 옵저버 클래스가 되기 위해 구현해야 하는 인터페이스를 가진다. 
  * `ConcreteObserverA`, `ConcreteObserverB`: 옵저버 인터페이스를 구현한 클래스이다. 각 옵저버는 특정 주제 객체에 등록해서 연락을 받을 수 있다.

### 옵저버 패턴 예시

* 날씨에 대한 정보를 가지는 `WeatherData` 객체가 있고 날씨 정보를 이용해서 정보를 표현하는 화면 객체들이 있다. 날씨 정보가 업데이트되면 각각 자신이 표현하고자 하는 정보를 화면에 띄우고자 한다.
  * `WeatherData` 를 주제 객체로, 화면 객체들을 옵저버로 구현할 수 있다.

```java
public interface Subject {
  public void registerObserver(Observer O);
  public void removeObserver(Observer O);
  public void notifyObservers();
}
```

* 주제 객체는 옵저버로 등록하거나 탈퇴하는 메소드를 가지며, 옵저버들에게 연락을 돌리는 메소드를 가진다.

```java
public interface Observer {
  public void update(float temperature, float humidity, float pressure);
}
```

* 옵저버들은 연락받은 정보로 자신의 디스플레이에 맞는 정보를 업데이트한다.

```java
public class WeahterData implements Subject {
  private ArrayList observers;
  private float temperature;
  private float humidity;
  private float pressure;
  
  public WeatherData() {
    observers = new ArrayList();
  }
  
  public void registerObserver(Observer o) {
    observers.add(o);
  }
  
  public void removeObserver(observer o){
    int i = observers.indexOf(o);
    if(i >= 0) {
      observers.remove(i);
    }
  }
  
  public void notifyObservers(){
    observers.forEach(o -> o.update(temperature, humidity, pressure));
  }
  
  public void measurementsChanged() {
    notifyObservers();
  }
  
  // setter, ...
}
```

* 상태에 변화가 생기면 `Observer` 인터페이스를 구현한, 주제 객체에 등록되어 있는 모든 객체에게 이 사실을 알릴 수 있게 된다.

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
  private float temperature;
  private float humidity;
  private Subject WeatherData;
  
  public CurrentConditionsDisplay(Subject weatherData) {
    this.weatherData = weatherData;
    weatherData.registerObserver(this);
  }
  
  public void update(float temperature, float humidity, float pressure) {
    this.temperature = temperature;
    this.humidity = humidity;
    display();
  }
  
  public void display() {
    // ...
  }
}
```

* 위의 예시 옵저버 외에도 날씨 정보를 필요로 하는 옵저버 객체가 있다면 옵저버 인터페이스를 구현 후 주제 객체에 등록만 하면 원하는 정보를 얻을 수 있다.
* 그런데 위의 예시는 `pressure` 정보를 필요로 하지 않지만 해당 정보를 받을 수밖에 없다. 필요한 정보만 받고 싶은 경우(pull) 주제 객체에 `get` 메소드를 추가하고 `update` 메소드에서 주제 객체의 참조 정보를 사용해서 필요한 정보만을 가져올 수 있다.
  * 필요한 정보마다 메소드를 호출해야 하는 불편함이 있을 수 있다.
  * 이러한 pull 방식과 기존의 push 방식을 모두 사용할 수 있는 자바 내장 옵저버 패턴이 존재한다.

#### 자바 내장 기능

```java
import java.util.Observable;
import java.util.Observer;

public class WeahterData extends Observable {
  private float temperature;
  private float humidity;
  private float pressure;
  
  public WeatherData() { }
  
  public void measurementsChanged() {
    setChanged();
    notifyObservers();
  }
  
  public void setMeasurements(float temperature, float humidity, float pressure) {
    this.temperature = temperature;
    this.humidity = humidity;
    this.pressure = pressure;
    measurementsChanged();
  }
 
  // getter, ...
 
}
```

* `setChanged`는 내부 플래그를 `true`로 만들어 알림이 동작하도록 하는 메소드이다. 또한 알림을 전송할 때 `synchronized` 를 보장한다. 
  * `Observable` 클래스는 내부에 옵저버들을 벡터로 가지고 있는데, 벡터를 꺼내고 저장하는 과정은 동기화가 필요하지만 옵저버들에게 연락하는 것은 동기화가 필요하지 않다.
  * 따라서 벡터를 꺼내고 저장하는 과정은 `synchronized` 를 통해 동기화를 보장해 준다. 

```java
import java.util.Observable;
import java.util.Observer;

public class CurrentConditionsDisplay implements Observer, DisplayElement {
  private float temperature;
  private float humidity;
  private Observable observable;
  
  public CurrentConditionsDisplay(Observable observable) {
    this.observable = observable;
    observable.addObserver(this);
  }
  
  public void update(Observable obs, Object arg) {
    if (obs instanceof WeatherData) {
      WeatherData weatherData = (WeatherData) obs;
      this.temperature = weatherData.getTemperature(); // using getter
      this.humidity = weatherData.getHumidity();
      display();
    }
  }
  
  public void display() {
    // ...
  }
}
```

* 자바 내장 기능을 사용해 위와 같이 필요한 데이터만 가져오도록 할 수 있다. 
* 편리한 기능이지만 `Observable` 은 인터페이스가 아닌 클래스이므로 다중 상속을 지원하지 않는 자바에서는 활용에 한계가 있다.

##### 참고; `Observable`과 `Observer`는 Java SE9 버전부터 Deprecated 되었다.

* 문서에서는 그 이유를 다음과 같이 설명하고 있다.
  * `Observer`와 `Observable`이 제공하는 이벤트 모델이 제한적이다.
  * 알림은 순서를 보장할 수 없으며 상태 변경은 1:1로 일치하지 않는다.
  * 더 풍부한 이벤트 모델을 `java.beans` 패키지가 제공한다.
  * 멀티 스레드에서의 신뢰할 수 있고 순서가 보장된 메시징은 `java.util.concurrent` 패키지의 자료 구조들 중 하나를 골라 쓰는 편이 낫다.
  * Reactive stream 스타일 프로그래밍은 `Flow` API를 쓰는 것을 권한다.

* 결론은 다른 더 좋은 대체제가 많으니 그것들을 써라, 정도인 것 같다. 실제로 구현할 일이 생기면 위에 언급된 것들을 잘 활용하는 것이 좋겠다.
