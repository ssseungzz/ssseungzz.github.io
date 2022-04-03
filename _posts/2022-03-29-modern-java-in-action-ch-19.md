---
title: "Modern Java in Action 19장; 함수형 프로그래밍 기법"
categories:
  - Language
tags:
  - java
toc: true
toc_label: "Contents"
toc_sticky: true
---

### 19. 함수형 프로그래밍 기법
 
#### 19.1 함수는 모든 곳에 존재한다
* 일반값처럼 취급할 수 있는 함수를 **일급 함수**라고 한다. 자바 8은 이전 버전과 달리 일급 함수를 지원한다.
  * `::` 연산자로 메서드 참조를 만들거나 람다 표현식으로 직접 함숫값을 표현해서 메서드를 함숫값으로 사용할 수 있다.

##### 19.1.1 고차원 함수
* 함숫값을 자바 8 스트림 처리 연산으로 전달하거나 메서드에 함숫값으로 메서드 참조를 전달해서 동적 파라미터화를 달성하는 용도로 사용하는 것 외에 함수를 인수로 받아 다른 함수로 반환하는 정적 메서드(ex - `Comparator.comparing`)도 있다.
* 다음 중 하나 이상의 동작을 수행하는 함수를 **고차원 함수**라고 부른다.
  * 하나 이상의 함수를 인수로 받음
  * 함수를 결과로 반환
* 자바 8에서는 함수를 인수로 전달할 수 있을 뿐 아니라 결과로 반환하고 지역 변수로 할당하거나 구조체로 삽입할 수 있으므로 자바 8의 함수도 고차원 함수라고 할 수 있다.
  * 고차원 함수를 적용할 때도 함수는 부작용이 없어야 하며 부작용을 포함하는 함수를 사용하면 문제가 발생한다는 규칙을 따라야 한다.
  * 고차원 함수나 메서드를 구현할 때 어떤 인수가 전달될지 알 수 없으므로 인수가 부작용을 포함할 가능성을 염두에 두어야 한다.

##### 19.1.2 커링
```java
static double converter(double x , double f, double b) {
  return x * f + b;
}
```
* 위와 같은 변환기를 만들면 매번 변환 요소와 기준치를 넣어야 한다. 혹은 각각을 변환하는 메서드를 따로 만드는 방법이 있지만 로직을 재활용할 수 없다.
* 커링의 개념을 적용해서 한 개의 인수를 갖는 변환 함수를 생산하는 팩토리를 정의할 수 있다.

```java
static DoubleUnaryOperator curriedConverter(double f, double b) {
  return (double x) -> x * f + b;
}

DoubleUnaryOperator convertUSDtoGBP = curriedConverter(0.6, 0);
double gbp = convertUSDtoGBP.applyAsDouble(1000);
```

* 커링은 x와 y라는 두 인수를 받는 함수 f를 한 개의 인수를 받는 g라는 함수로 대체하는 기법이다.
  * 이때 g라는 함수 역시 하나의 인수를 받는 함수를 반환한다. 함수 g와 원래 함수 f가 최종적으로 반환하는 값은 같다. 즉, f(x, y) = (g(x))(y)가 성립한다.
  * 이 과정을 일반화할 수 있다. 예를 들어 여섯 개의 인수를 가진 함수를 커리해서 우선 2, 4, 6번째 인수를 받아 5번째 인수를 받는 함수를 반환하고 다시 이 함수는 남은 1,3 번째 인수를 받는 함수를 반환한다. 이와 같은 여러 과정이 끝까지 완료되지 않은 상태를 가리켜 '함수가 부분적으로 적용되었다'라고 말한다.

#### 19.2 영속 자료구조
* 여기서 말하는 영속은 데이터베이스에서 프로그램 종료 후에도 남아 있음을 의미하는 영속과는 다르다.
* 함수형 메서드에서는 전역 자료구조나 인수로 전달된 구조를 갱신할 수 없다. 바꾼다면 같은 메서드를 두 번 호출했을 때 결과가 달라지면서 참조 투명성에 위배되고 인수를 결과로 매핑할 수 있는 능력이 상실된다.

##### 19.2.1 파괴적인 갱신과 함수형
* 파괴적인 갱신이란 기존의 자료구조를 변경하는 것을 의미한다. 이 경우 기존의 자료구조에 의존하던 사용자들은 혼란에 빠질 수 있다.
* 함수형에서는 계산 결과를 표현할 자료구조가 필요하면 기존의 자료구조를 갱신하는 것이 아니라 새로운 자료구조를 만들도록 한다.

##### 19.2.2 트리를 사용한 다른 예제
* 이진 탐색 트리를 사용한다고 가정해 보자.
```java
class Tree {
  private String key;
  private int val;
  private Tree left, right;
  public Tree(String k, int v, Tree l, Tree r) {
    key = k; val = v; left = l; right = r;
  }
}

class TreeProcessor {
  public static int lookup(String k, int defaultval, Tree t) {
    if (t == null) return defaultVal;
    if (k.equals(t.key)) return t.val;
    return lookup(k, defaultVal, k.compareTo(t.key) < 0 ? t.left : t.right);
  }

  public static void update(String k, int newVal, Tree t) {
    if (t == null) { /* 새 노드를 추가해야 함 */ }
    else if (k.equals(t.key)) t.val = newVal;
    else update(k, newVal, k.compareTo(t.key) < 0 ? t.left : t.right);
  }
}
```
* 새로운 노드를 추가할 때 `update` 메서드가 탐색한 트리를 그대로 반환하는 게 가장 쉬운 방법이지만 사용자가 `update` 시 즉석에서 트리를 갱신할 수 있으며 전달한 트리가 그대로 반환된다는 사실, 원래 트리가 비어 있으면 새로운 노드가 반환될 수 있다는 사실을 모두 기억해야 한다.


```java
public static Tree update(String k, int newVal, Tree t) {
  if (t == null) {
    t = new Tree(k, newVal, null, null);
  }
  else if (k.equals(t.key)) {
    t.val = newVal;
  }
  else if (k.compareTo(t.key) < 0) {
    t.left = update(k, newVal, t.left);
  }
  else {
    t.right = update(k, newVal, t.right);
  }
  return t;
}
```

* 두 `update` 모두 기존 트리를 변경한다.

##### 19.2.3 함수형 접근법 사용
* 이 문제를 함수형으로 처리하기 위해서는 새로운 키/값 쌍을 저장할 새로운 노드를 만들어야 하며 트리의 루트에서 새로 생성한 노드의 경로에 있는 노드들도 새로 만들어야 한다.
```java
public static Tree fupdate(String k, int newval, Tree t) {
  return (t == null) ?
          new Tree(k, newval, null, null) :
          k.equals(t.key) ?
            new Tree(k, newval, t.left, t.right) :
          k.compareTo(t.key) < 0 ?
            new Tree(t.key, t.val, fupdate(k, newval, t.left), t.right) :
            new Tree(t.key, t.val, t.left, fupdate(k, newval, t.right))l
}
```
* 위 함수에서는 새로운 트리를 만든다. 하지만 인수를 이용해서 가능한 한 많은 정보를 공유한다.
  * 이와 같은 함수형 자료구조를 **영속**이라고 하며 프로그래머는 인수로 전달된 자료구조를 함수가 변화시키지 않을 것이라는 사실을 확신할 수 있다.
  * 필드를 `final`로 선언함으로써 기존 구조를 변화시키지 않는다는 규칙을 강제할 수 있지만 `final`은 필드에만 적용되며 객체에는 적용되지 않으므로 각 객체의 필드에 적절하게 사용해야 한다.

![](https://user-images.githubusercontent.com/55083845/160835132-af02dbb6-d01a-4ed9-80b9-ca7f896647d0.jpeg)
* 일부 사용자만 볼 수 있게 갱신하면서도 다른 일부 사용자는 이를 알아차릴 수 없게 하고 싶다면 두 가지 방법이 있다.
  * 고전적인 자바 해법으로 어떤 값을 갱신할 때 먼저 복사해야 하는지 확인한다.
  * 함수형 해법은 갱신을 수행할 때마다 논리적으로 새로운 자료구조를 만든 다음에 사용자에게 적절한 버전의 자료구조를 전달할 수 있다. API로 이 방법을 강제할 수 있다. 사용자가 볼 수 있도록 갱신을 수행해야 한다면 최신 버전을 반환하는 API를 사용한다. 보이지 않도록 수행해야 한다면 인수를 복사한 값을 반환한다.

#### 19.3 스트림과 게으른 평가
* 스트림은 단 한 번만 소비할 수 있다는 제약이 있어서 재귀적으로 정의할 수 없다.
##### 19.3.1 자기 정의 스트림
* 소수를 생성하는 예제로 재귀 스트림을 살펴본다.
  * 이론적으로 소수로 나눌 수 있는 모든 수는 제외할 수 있다.
###### 1단계: 스트림 숫자 얻기
```java
static IntStream numbers() {
  return IntStream.iterate(2, n -> n + 1);
}
```
###### 2단계: 머리 획득
```java
static int head(IntStream numbers) {
  return numbers.findFirst.getAsInt();
}
```
###### 3단계: 꼬리 필터링

```java
static IntStream tail(IntStream numbers) {
  return numbers.skip(1);
}

IntStream numbers = numbers();
int head = head(numbers);
IntStream filtered = tail(numbers).filter(n -> n % head != 0);
```

###### 4단계: 재귀적으로 소수 스트림 생성
```java
static IntStream primes(IntStream numbers) {
  int head = head(numbers);
  return IntStream.concat(
      Intstream.of(head),
      primes(tail(numbers).filter(n -> n % head != 0))
  );
}
```
* 마지막 단계 코드를 실행하면 에러라 발생한다. `findFirst`와 `skip`은 최종 연산이므로 스트림에 호출하면 스트림이 완전 소비되기 때문이다.
* 게다가 정적 메서드 `IntStream.concat`은 두 개의 스트림 인스턴스를 인수로 받는다. 두 번째 인수가 `primes`를 직접 재귀적으로 호출하면서 무한 재귀에 빠진다.
  * 두 번째 인수에서 `primes`를 게으르게 평가하는 방식으로 문제를 해결할 수 있다. 즉, 소수를 처리할 필요가 있을 때만 스트림을 실제로 평가한다.

##### 19.3.2 게으른 리스트 만들기
* 자바 8의 스트림은 요청할 때만 값을 생성하는 블랙박스와 같다. 스트림에 일련의 연산을 적용하면 연산이 수행되지 않고 일단 저장된다.
* 스트림에 최종 연산을 적용해서 실제 계산을 해야 하는 상황에서만 실제 연산이 이루어진다.
* 좀 더 일반적인 스트림의 형태에는 게으른 리스트가 있다. 게으른 리스트는 고차원 함수라는 개념도 지원한다. 함숫값을 자료구조에 저장해서 함숫값을 사용하지 않은 상태로 보관할 수 있다. 하지만 저장한 함숫값을 호출(즉, 요청)하면 더 많은 자료구조를 만들 수 있다.

![](https://user-images.githubusercontent.com/55083845/161092530-be3b3855-7dfb-454f-987f-0aad6dc042df.jpeg)

###### 기본적인 연결 리스트
```java
interface MyList<T> {
  T head();

  MyList<T> tail();

  default boolean isEmpty() {
    return true;
  }
}
```
```java
class MyLinkedList<T> implements MyList<T> {
  private final T head;
  private final MyList<T> tail;
  public MyLinkedList(T head, MyList<T> tail) {
    this.head = head;
    this.tail = tail;
  }

  public T head() {
    return head;
  }

  public MyList<T> tail() {
    return tail;
  }

  public boolean isEmpty() {
    return false;
  }
}

class Empty<T> implements MyList<T> {
  public T head() {
    throw new UnsupportedOperationException();
  }

  public MyList<T> tail() {
    throw new UnsupportedOperationException();
  }
}

MyList<Integer> l = new MyLinkedList<>(5, new MyLinkedList<>(10, new Empty<>()));
```

###### 기본적인 게으른 리스트
```java
class LazyList<T implements MyList<T> {
  final T head;
  final Supplier<MyList<T>> tail;
  public LazyList(T head, Supplier<MyList<T>> tail) {
    this.head = head;
    this.tail = tail;
  }

  public T head() {
    return head;
  }

  public MyList<T> tail() {
    return tail.get(); // Supplier로 게으른 동작을 만들었다.
  }

  public boolean isEmpty() {
    return false;
  }
}
```
* `Supplier`의 `get`을 호출하면 노드가 만들어진다.

###### 소수 생성으로 돌아와서
```java
public static MyList<Integer> primes(MyList<Integer> numbers) {
  return new LazyList<>(
    numbers.head(),
    () -> primes(
      numbers.tail()
             .filter(n -> n % numbers.head != 0)
    )
  );
} 

public MyList<T> filter(Predicate<T> p) {
  return isEmpty() ?
         this :
         p.test(head()) ?
              new LazyList<>(head(), () -> tail().filter(p)) :
              tail.filter(p);
}
```

* 자바 8 덕분에 함수를 자료구조 내부로 추가할 수 있으며 이런 함수는 자료구조를 만드는 시점이 아니라 요청 시점에 실행된다.
* 게으른 리스트는 자바 8 기능 스트림과의 연결고리를 제공한다.
* 적극적으로 기능을 실행하는 것보다는 게으른 편이 성능이 좋다고 가정할 수 있지만, 자료구조의 10퍼센트 미만의 데이터만 활용하는 상황에서는 게으른 실행으로 인한 오버헤드가 더 커질 수 있다.
  * 또한 자료구조 내부에 추가된 함수가 반복적으로 호출될 수 있다. 이는 처음 요청을 호출할 때만 함수가 호출되도록 하고 결과값을 캐시해서 해결할 수 있다.

#### 19.4 패턴 매칭
* 함수형 프로그래밍을 구분하는 또 하나의 중요한 특징으로 (구조적인) 패턴 매칭을 들 수 있다. (정규표현식과는 다르다.)

##### 19.4.1 방문자 디자인 패턴
* 자바에서는 방문자 디자인 패턴으로 자료형을 언랩할 수 있다. 특히 특정 데이터 형식을 '방문'하는 알고리즘을 캡슐화하는 클래스를 따로 만들 수 있다.
* 방문자 클래스는 지정된 데이터 형식의 인스턴스를 입력으로 받는다. 그리고 인스턴스의 모든 멤버에 접근한다.

```java
class BinOp extends Expr {
  ...
  public Expr accept(SimplifyExprVisitor v) {
    return v.visit(this);
  }
}

public class SimplifyExprVisitor {
  ...
  public Expr visit(BinOp e) { // SimplifyExprVisitor는 BinOp 객체를 언랩할 수 있다.
    if ("+".equals(e.opname) && e.right instance of Number && ...) {
      return e.left;
    }
    return e;
  }
}
```

##### 19.4.2 패턴 매칭의 힘

```scala
def simplifyExpression(expr: Expr): Expr = expr match {
  case BinOp("+", e, Number(0)) => e
  case BinOp("*", e, Number(1)) => e
  case BinOp("/", e, Number(1)) => e
  case => expr
}
```


* 트리와 비슷한 자료구조를 다룰 때 이와 같은 패턴 매칭을 사용하면 간결하고 명확한 코드를 구현할 수 있다. 특히 컴파일러를 만들거나 비즈니스 규칙 처리 엔진을 만들 때 유용하.
* 자바 8의 람다를 이용하면 패턴 매칭과 비슷한 코드를 만들 수 있다.

###### 자바로 패턴 매칭 흉내 내기
* 자바 8의 람다를 이용한 패턴 매칭 흉내는 단일 수준의 패턴 매칭만 지원한다.
  * 람다를 이용하며 코드에 if-then-else가 없어야 한다. '조건 ? e1 : e2'와 메서드 호출로 if-then-else를 대신할 수 있다.

```java
myIf(condition, () -> e1, () -> e2);

static <T> T myIf(boolean b, Supplier<T> truecase, Supplier<T> falsecase) {
  return b ? truecase.get() : falsecase.get();
}
```

* 람다를 이용하면 단일 수준의 패턴 매치을 간단하게 표현할 수 있으므로 여러 개의 if-then-else 구분이 연결되는 상황을 깔끔하게 정리할 수 있다.


```java
interface TriFunction<S, T, U, R> {
  R apply(S s, T t, U u);
}

static <T> T patternMatchExpr(
                        Expr e,
                        TriFunction<String, Expr, Expr, T> binopcase,
                        Fuction<Integer, T> numcase,
                        Supplier<T> defaultcase){
  return 
    (e instanceof BinOp) ?
      binopcase.apply(((Binop)e).opname, ((Binop)e).left, ((Binop)e).right) :
    (e instanceof Number) ?
      numcase.apply((Number)e.val) :
      defaultcase.get()
}
```


```java
public static Expr simplify(Expr e) {
  TriFunction<String, Expr, Expr, Expr> binopcase = 
    (opname, left, right) -> { // Binop 표현식 처리
      if ("+".equals(opname)) {
        if (left instnaceof Number && ((Number)left).val == 0) {
          return right;
        }
        if (right instnaceof Number && ((Number)right).val == 0) {
          return left;
        }
      }
      if ("*".equals(opname)) {
        if (left instnaceof Number && ((Number)left).val == 1) {
          return right;
        }
        if (right instnaceof Number && ((Number)right).val == 1) {
          return left;
        }
      }
      return new Binop(opname, left, right);
    };
  Function<Integer, Expr> numcase = val -> new Number(val) // 숫자 처리
  Supplier<Expr> defaultcase = () -> new Number(0) // 기본 처리

  return patterMatchExpr(e, binopcase, numcase, defaultcase); // 패턴 매칭 적용
}
```


#### 19.5 기타 정보

##### 19.5.1 캐싱 또는 기억화
* 기헉화는 메서드에 래퍼로 캐시를 추가하는 기법이다. 래퍼가 호출되면 인수, 결과 쌍이 캐시에 존재하는지 먼저 확인한다.
  * 캐시에 값이 존재하면 저장된 값을 즉시 반환한다.
  * 존재하지 않으면 메서드를 호출해서 결과를 계산한 다음에 새로운 인수, 결과 쌍을 캐시에 저장하고 결과를 반환한다.
  * 캐싱, 즉 다수의 호출자가 공유하는 자료구조를 갱신하는 기법이므로 순수 함수형 해결 방식은 아니지만 감싼 버전의 코드는 참조 투명성을 유지할 수 있다.


```java
final Map<Range, Integer> numberOfNodes = new HashMap<>();
Integer computeNumberOfNodesUsingCache(Range range) {
  Integer result = numberOfNodes.get(range);
  if (result != null) {
    return result;
  }
  result = computeNumberOfNodes(range);
  numberOfNodes.put(range, result);
  return result;
}
```

* `computeNumberOfNodesUsingCache`는 참조 투명성을 갖는다. (`computeNumberOfNodes`도 참조 투명하다는 가정 하에)
  * 하지만 `numberOfNodes`는 공유된 가변 상태며 해시 맵은 동기화되지 않았으므로 스레드 안전성이 없다. 잠금으로 보호되거나 잠금 없이 동시 실행을 지원하는 다른 자료구조를 사용할 수 있지만 성능이 저하될 수 있다. 맵에서 찾는 과정과 인수, 결과 쌍을 맵에 추가하는 동작 사이에서 레이스 컺ㄴ디션이 발생하기 때문이다.
  * 즉, 여러 프로세스가 같은 값을 맵에 추가하기 위해 여러 번 계산하는 일이 발생할 수 있다.
* 함수형 프로그래밍을 사용해서 동시성과 가변 상태가 만나는 상황을 완전히 없애는 게 가장 좋지만 캐싱 같은 저수준 성능 문제는 해결되지 않는다.
  * 캐싱을 구현할 것인지 여부와 별개로 코드를 함수형으로 구현한다면 호출하려는 메서드가 공유된 가변 상태를 포함하지 않음을 미리 알 수 있으므로 동기화 등을 신경 쓸 필요가 없어진다.

##### 19.5.3 '같은 객체를 반환함'은 무엇을 의미하는가?
* 참조 투명성이란 인수가 같다면 결과도 같아야 한다는 규칙을 만족함을 의미한다. 
  * 같은 주소값을 갖지 않는다면 참조 투명성을 갖지 않는다고 결론내릴 수 있으나 자료구조를 변경하지 않는 상황에서 참조가 다르다는 것은 큰 의미가 없으며 주소값이 달라도 논리적으로는 같다고 판단할 수 있다.
* 일반적으로 함수형 프로그래밍에서는 데이터가 변경되지 않으므로 같다는 의미는 `==`(참조가 같음)이 아니라 구조적인 값이 같다는 것을 의미한다.

##### 19.5.3 콤비네이터
* 함수형 프로그래밍에서는 두 함수를 인수로 받아 다른 함수를 반환하는 등 함수를 조합하는 고차원 함수를 많이 사용하게 된다. 이처럼 함수를 조합하는 기능을 콤비네이터라 부른다.


```java
static <A, B, C> Function<A, C> compose(Function<B, C> g, Function<A, B> f) {
  return x -> g.apply(f.apply(x));
}
```
