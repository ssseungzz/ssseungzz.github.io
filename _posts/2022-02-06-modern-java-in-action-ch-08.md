---
title: "Modern Java in Action 8장; 컬렉션 API 개선"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 8. 컬렉션 API 개선

#### 8.1 컬렉션 팩토리
###### UnsupportedOperationException 발생
* `Arrays.asList()`와 같은 팩토리 메서드를 이용해서 배열을 만드는 경우 내부적으로 고정된 크기의 배열이 되므로 요소를 추가할 수 없다.
* 집합의 경우 배열을 생성자에 제공하는 방법이나 스트림 API를 이용해서 생성할 수 있는데 내부적으로 불필요한 객체 할당을 필요로 한다.

##### 8.1.1 리스트 팩토리
* `List.of` 팩토리 메소드를 이용하면 리스트를 만들 수 있다.
* 변경할 수 없는 리스트가 만들어지므로 요소의 추가가 불가하며, `set()` 메서드로 아이템을 바꾸려 해도 `UnsupportedOperationException`이 발생한다.
* 요소 자체가 변하는 것을 막을 수 있는 방법은 없으며 `null` 요소는 금지한다.
> 다중 요소를 받을 수 있는 오버로드 버전은 없는데, 내부적으로 가변 인수 버전은 추가 배열을 할당해서 리스트로 감싸므로 배열을 할당하고 초기화하며 나중에 가비지 컬렉션을 하는 비용을 제거할 수 있다.

##### 8.1.2 집합 팩토리
* `Set.of`로 만들 수 있으며 오직 고유의 요소만 포함 가능하다.

##### 8.1.3 맵 팩토리
* `Map.of` 팩토리 메서드에 키와 값을 번갈아 제공하는 방법으로 맵을 만들 수 있다.
```java
Map<String, Integer> ageOfFriends = Map.of("kim", 20, "Lee", 24, "Park", 31);
```
* 쌍이 조금 더 많아질 경우 `Map.Entry<K, V>` 객체를 인수로 받으며 가변 인수로 구현된 `Map.ofEntries` 팩토리 메서드를 이용하는 것이 좋다.
    * 이 메서드는 키와 값을 감쌀 추가 객체 할당을 필요로 한다.
    * `Map.entry`는 `Map.Entry` 객체를 만드는 새로운 팩토리 메서드다.

#### 8.2 리스트와 집합 처리
* 자바 8에서는 `List`, `Set` 인터페이스에 다음과 같은 메서드를 추가했다.
    * `removeIf`: 프레디케이트를 만족하는 요소를 제거한다.
    * `replaceAll`: 리스트에서 이용할 수 있는 기능으로 `UnaryOperator` 함수를 이용해 요소를 바꾼다.
    * `sort`: `List` 인터페이스에서 제공하는 기능으로 요소를 정렬한다.

##### 8.2.1 removeIf 메서드
* 내부적으로 for-each 루프는 `Iterator` 객체를 사용한다
    * `Iterator` 개체는 `next()`, `hasNext()`를 통해 소스를 질의한다.
    * `Collection` 객체 자체는 메서드를 호출해 요소에 특정 작업을 한다.
    * `Collection` 객체에 for-each를 사용해 요소를 `remove()` 할 때 두 개의 개별 객체가 컬렉션을 관리하게 되고, 결과적으로 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않는다.
    * `Iterator` 객체를 명시적으로 사용하고 그 객체의 `remove()`를 호출함으로써 서로 동기화되지 않는 것에서 오는 문제를 해결할 수 있다.
* 이 코드 패턴은 `removeIf()` 메서드로 바꿀 수 있다. 

##### replaceAll 메서드
* 스트림 API를 사용하면 리스트의 각 요소를 새로운 요소로 바꿀 수 있지만 새 문자열 컬렉션을 만들게 된다.
* 기존 컬렉션을 바꾸기 위해 for-each 루프를 사용하면 위와 똑같은 문제를 겪을 수 있다.
* 이 대신 `replaceAll`을 사용하면 간단하게 기존 컬렉션의 요소를 바꿀 수 있다.

#### 8.3 맵 처리

##### 8.3.1 forEach 메서드
* `BiConsumer`(키와 값을 인수로 받음)를 인수로 받는 `forEach` 메서드로 맵에서 키와 값을 반복하면서 확인할 수 있다.

##### 8.3.2 정렬 메서드
* `Entry.comparingByValue`, `Entry.comparingByKey`를 통해 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있다.

##### 8.3.3 getOrDefault 메서드
* 찾으려는 키가 존재하지 않으면 `getOrDefault` 메서드를 통해 기본값을 반환받을 수 있다.
    * 첫 번째 인수로 키를, 두 번째 인수로 기본값을 받는다. 키가 존재하더라도 값이 널인 상황에서는 `getOrDefault`가 널을 반환할 수 있다.

##### 8.3.4 계산 패턴
* 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장하기 위해 아래의 메서드들을 사용할 수 있다.
    * `computeIfAbsent`: 제공된 키에 해당하는 값이 없으면 키를 이용해 새 값을 계산하고 맵에 추가한다.
    * `computeIfPresent`: 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
    * `compute`: 제공된 키로 새 값을 계산하고 맵에 저장한다.

##### 8.3.5 삭제 패턴
* `remove(key, value)`를 통해 간단하게 구현할 수 있다.

##### 8.3.6 교체 패턴
* 맵의 항목을 바꾸는 두 개의 메서드가 추가되었다.
    * `replaceAll`: `BiFunction`을 적용한 결과로 각 항목의 값을 교체한다.
    * `Replace`: 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

##### 8.3.7 합침
* 두 개의 맵을 합치는 경우 `putAll`을 사용할 수 있다. 중복된 코드가 없다면 잘 동작한다.
* `merge` 메서드는 중복된 키를 어떻게 합칠지 결정하는 `BiFunction`을 인수로 받는다.
```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) -> 
    everyone.merge(k, v, (movie1, movie2) -> movie1 + "&" + movie2));
```
* `merge`는 저장된 키와 연관된 값이 없거나 널이면 키를 널이 아닌 값과 연결한다. 아니면 연결된 값을 주어진 매핑 함수의 결과 값으로 대치하거나 결과가 널이면 항복을 제거한다.

#### 8.4 개선된 ConcurrentHashMap
* `ConcurrentHashMap`은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용해서 동시성 친화적이다.

##### 8.4.1 리듀스와 검색
* `forEach`: 각 (키, 값) 쌍에 주어진 액션 실행
* `reduce`: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
* `search`: 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수 적용
* 키에 함수 받기, 값, `Map.Entry`, (키, 값) 인수를 이용한 네 가지 연산 형태를 지원한다.
    * 키, 값으로 연산(`forEach`, `reduce`, `search`)
    * 키로 연산(`forEachKey`, `reduceKeys`, `searchKeys`)
    * 값으로 연산(`forEachValue`, `reduceValues`, `searchValues`)
    * `Map.Entry` 객체로 연산(`forEachEntry`, `reduceEntries`, `searchEntries`)
* 이들 연산은 맵의 상태를 잠그지 않고 연산을 수행하므로 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.
* 연산에 병렬성 기준값을 지정해야 한다. 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 진행하고, 1로 지정하면 공통 스레드 풀을 이용해 병렬성을 극대화한다. `Long.MAX_VALUE`로 설정하면 한 개의 스레드로 연산을 실행한다.

##### 8.4.2 계수
* `mappingCount` 메서드는 맵의 매핑 개수를 반환한다.

##### 8.4.3 집합뷰
* `ConcurrentHashMap`을 집합 뷰로 반환하는 `keySet`이라는 메서드를 지원한다.
* 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다. `newKeySet` 메서드를 통해 `ConcurrentHashMap`으로 유지되는 집합을 만들 수도 있다.