---
title: "Modern Java in Action 7장; 병렬 데이터 처리와 성능"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 7. 병렬 데이터 처리와 성능

#### 7.1 병렬 스트림
* 컬렉션에 `parallelStream`을 호출하면 병렬 스트림이 생성된다.
    * 각각의 스레드에서 처리할 수 있도록 스트림 요소를 청크로 분할한 스트림이다.
    * 병렬 스트림을 이용하면 모든 멀티코어 프로세스가 각각의 청크를 처리하도록 할당할 수 있다.

##### 7.1.1 순차 스트림을 병렬 스트림으로 반환하기
* 순차 스트림에 `parallel` 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다.
    * 스트림이 여러 청크로 분할되어 병렬로 수행된다. 마지막으로 리듀싱 연산으로 생성된 부분 결과를 다시 리듀싱 연산으로 합쳐 결과를 도출한다.
* `sequential`로 병렬 스트림을 순차 스트림으로 바꿀 수 있다.

> 병렬 스트림은 내부적으로 `ForkJoinPool`을 사용한다. 이 풀은 기본적으로 프로세서 수에 상응하는 스레드를 갖는다.

##### 7.1.2 스트림 성능 측정
* JMH 라이브러리를 이용해 작은 벤치마크를 구현해서 확인할 수 있다.
    * 간단하고, 어노테이션 방식을 지원하며 안정적으로 JVM 언어용 벤치마크를 구현할 수 있다.
* 벤치마크가 가능한 가비지 컬렉터의 영향을 받지 않도록 힙의 크기를 충분히 설정하고 벤치마크가 끝날 때마다 가비지 컬렉터가 실행되도록 강제하는 등의 방식으로 결과의 신뢰도를 높일 수 있다.
* 반복 결과로 박싱된 객체가 만들어져서 언박싱해야 하는 경우, 반복 작업을 병렬로 수행할 수 있는 독립 단위로 나누기가 어려운 경우(가령, 이전 연산의 결과에 따라 다음 함수의 입력이 달라지는 경우) 등에는 성능 향상을 기대하기 어렵다.

###### 더 특화된 메서드 사용
* 멀티코어 프로세서를 활용해 효과적으로 합계 연산을 병렬로 실행하려면 언박싱 오버헤드를 줄이고 쉽게 분할할 수 있는 숫자 범위를 사용할 수 있어야 한다.
    * `LongStream.rangeClosed`는 이를 지원한다.
* 올바른 자료구조를 선택해야 병렬 실행이 최적의 성능을 발휘할 수 있다.
* 그럼에도 병렬화를 이용하기 위해서는 스트림을 재귀적으로 분할하고 각 서브스트림을 서로 달느 스레드의 리듀싱 연산으로 할당하고 이들 결과를 하나의 값으로 합쳐야 하는 등의 비용이 존재하므로 코어 간에 데이터 전송 시간보다 훨씬 오래 걸리는 작업만 병렬로 다른 코어에서 수행하는 것이 바람직하다.

##### 7.1.3 병렬 스트림의 올바른 사용법
* 병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘의 사용 때문에 일어난다.
    * 다수의 스레드에서 동시에 데이터에 접근하는 문제가 발생할 수 있다.

##### 7.1.4 병렬 스트림 효과적으로 사용하기
* 확신이 서지 않으면 측정해서 확인하는 것이 낫다.
* 박싱을 주의한다. 되도록 기본형 특화 스트림을 사용하는 것이 좋다.
* 요소의 순서에 의존하는 연산을 병렬 스트림에서 수행하면 순차 스트림보다 성능이 떨어질 수 있다. 
* 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려한다. 하나의 요소를 처리하는 데에 드는 비용이 높다면 병렬 스트림으로 성능을 개선할 수 있는 가능성이 있다.
* 소량의 데이터에서는 병렬 스트림이 도움이 되지 않는다.
* 스트림을 구성하는 자료구조가 적절한지 확인한다. 가령, `ArrayList`는 요소를 탐색하지 않고도 분할할 수 있어 `LinkedList`보다 효율적으로 분할할 수 있다.
* 스트림의 특성과 파이프라인 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다. `SIZED` 스트림은 정확히 같은 크기의 두 스트림으로 분할할 수 있으므로 효과적으로 스트림을 병렬 처리할 수 있지만 필터 연산이 있으면 스트림의 길이를 예측할 수 없으므로 효과적으로 병렬 처리하기 어렵다.
* 최종 연산의 병합 과정이 비싸다면 병렬 스트림으로 얻은 성능의 이득이 상쇄될 수 있다.
* 병렬 스트림이 수행되는 내부 인프라구조도 살펴보아야 한다.

#### 7.2 포크/조인 프레임워크

##### 7.2.1 RecursiveTask 활용
* 스레드 풀을 이용하려면 `RecursiveTask<R>`의 서브 클래스를 만들어야 한다.
    * `R`은 병렬화된 태스크가 생성하는 결과 형식 또는 결과가 없을 때는 `RecursiveAction` 형식이다.
    * 추상 메서드 `compute`를 구현해야 한다. 이 메서드는 태스크를 서브 태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브 태스크의 결과를 생성할 알고리즘을 정의한다.
* 일반적으로 애플리케이션에서는 둘 이상의 `ForkJoinPool`을 사용하지 않는다. 필요한 곳에서 언제든 가져다 쓸 수 있도록 `ForkJoinPool`을 한 번만 인스턴스화해서 정적 필드에 싱글턴으로 저장한다.
    * `ForkJoinPool`을 만들면서 인수가 없는 디폴트 생성자를 이용했는데, 이는 JVM에서 이용할 수 있는 모든 프로세서가 자유롭게 풀에 접근할 수 있음을 의미한다.
    * 더 정확하게는 `Runtime.availableProcessors`의 반환값으로 풀에 사용할 스레드 수를 결정한다.

###### RecursiveTask<R>의 서브 클래스의 실행
* `RecursiveTask<R>`의 서브 클래스를 `ForkJoinPool`로 전달하면 풀의 스레드가 그 클래스의 `compute` 메서드를 실행하면서 작업을 수행한다.
    * 이 메서드는 병렬로 실행할 수 있을 만큼 태스크의 크기가 작아졌는지 확인하고 그렇지 않으면 분할해서 새로운 인스턴스로 할당한다. 
    * 주어진 조건을 만족할 때까지 태스크 분할을 반복하고 나면 서브태스크는 순차적으로 처리되며 포킹 프로세스로 만들어진 이진트리의 태스크를 루트에서 역순으로 방문한다. 각 서브태스크의 부분 결과를 합쳐서 최종 결과를 계산한다.

##### 7.2.2 포크/조인 프레임워크를 제대로 사용하는 방법
* `join` 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다. 따라서 두 서브태스크가 모두 시작된 다음에 `join`을 호출해야 한다. 그렇지 않으면 각각의 서브태스크가 다른 태스크가 끝나길 기다리는 일이 발생한다.
* `RecursiveTask` 내에서는 `ForkJoinPool`의 `invoke` 메서드를 사용하지 말아야 한다. 대신 `compute`나 `fork` 메서드를 직접 호출할 수 있다. 순차 코드에서 병렬 계산을 시작할 때만 `invoke`를 사용한다.
* 서브태스크에 `fork` 메서드를 호출해서 `ForkJoinPool`의 일정을 조절할 수 있다. 양쪽 작업에 `fork`를 호출하는 대신 한쪽에는 `compute`를 호출하면 두 서브태스크의 한 태스트에는 같은 스레드를 재사용할 수 있다.
* 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅이 어렵다. `fork`라 불리는 다른 스레드에서 `compute`를 호출하므로 스택 트레이스가 도움이 되지 않는다.
* 각 서브태스크의 실행 시간은 새로운 태스크르 포킹하는 데에 드는 시간보다 길어야 한다. JIT 컴파일러에 의해 최적화되려면 몇 차례의 준비 과정 또는 실행 과정을 거쳐야 한다. 성능 측정 시 여러 번 프로그램을 실행할 결과를 측정해야 한다. 컴파일러 최적화는 병렬 버전보다는 순차 버전에 집중될 수 있다.ㄴ

##### 7.2.3 작업 훔치기
* 포크/조인 프레임워크에서는 작업 훔치기라는 기법으로 `ForkJoinPool`의 모든 스레드를 거의 공정하게 분할한다.
    * 각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와 작업을 처리한다.
    * 이때 한 스레드는 다른 스레드보다 자신에게 할당된 태스크를 더 빨리 처리할 수 있는데, 할 일이 없어진 스레드는 유후 상태로 바뀌는 게 아니라 다른 스레드 큐의 꼬리에서 작업을 훔쳐온다. 모든 태스크가 작업을 끝낼 때까지, 즉 모든 큐가 빌 때까지 이 과정을 반복한다.
    * 따라서 태스크 크기를 작게 나누어야 작업자 스레드 간의 작업 부하를 비슷한 수준으로 유지할 수 있다.

#### 7.3 Spliterator 인터페이스
* `Spliterator`는 소스의 요소 탐색 기능을 제공하며 병렬 작업에 특화되어 있다.
* 자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 `Spliterator` 구현을 제공한다.

##### 7.3.1 분할 과정
* 1단계에서 첫 번째 `Spliterator`에 `trySplit`을 호출하면 두 번째 `Spliterator`가 생성된다.
* 2단계에서 두 개의 `Spliterator`에 `trySplit`을 다시 호출하면 네 개의 `Spliterator`가 생성된다.
* `trySplit`의 결과가 `null`이 될 때까지 이 과정을 반복한다.
* 4단계에서 `Spliterator`에 호출한 모든 `trySplit`의 결과가 `null`이면 재귀 분할 과정이 종료된다.
* 이 분할 과정은 `characteristics` 메서드로 정의하는 `Spliterator`의 특성에 영향을 받는다.

###### Spilterator 특정
* `characteristics` 메서드는 `Spliterator` 자체의 특성 집합을 포함하는 `int`를 반환한다.
    * 이 특성을 참고해서 더 잘 제어하고 최적화할 수 있다.

##### 7.3.2 커스텀 Spliterator 구현하기
* 순차 스트림을 병렬 스트림으로 바꿀 때 스트림 분할 위치에 따라 잘못된 결과가 나올 수 있다.
    * 이는 임의의 위치에서 분할하는 게 아닌, 특정 위치에서만 분할하는 방법으로 해결할 수 있다.

```java
class WordCounterSpliterator implements Spliterator<Character> {
    private final String string;
    private int currentChar = 0;
    
    public WordCounterSpliterator(String string) {
        this.string = string;
    }

    @Override
    public boolean tryAdvance(Consumer<? super Character> action) {
        action.accept(string.charAt(currentChar++));
        return currentChar < string.length();
    }

    @Override
    public Spliterator<Character> trySplit() {
        int currentSize = string.length() - currentChar();
        if (currentSize < 10) {
            return null;
        }
        for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
            if(Character.isWhitespace(string.charAt(splitPos))) {
                Spliterator<Character> spliterator =
                    new WordCounterSpliterator(string.subString(currentChar, splitPos));
                currentChar = splitPos;
                return spliterator;
            }
        }
        return null;
    }

    @Override
    public long estimateSize() {
        return string.length() - currentChar;
    }

    @Override
    public int characteristics() {
        return ORDERED + SIZED + SUBSIZED + NON-NULL + IMMUTABLE;
    }
}
```
* `trySplit`은 반복될 자료구조를 분할하는 로직으로 가장 중요한 메서드다. 우선 분할 동작을 중단할 한계를 설정하고 그에 따라 분할을 중지하거나 분할이 필요한 경우 정해진 기준으로 분할하도록 한다. 분할할 위치를 찾았으면 새로운 `Spliterator`를 만든다.
* `characteristics` 메서드는 프레임워크에 `Spliterator`가 어떤 특성을 가졌는지 알려 준다.