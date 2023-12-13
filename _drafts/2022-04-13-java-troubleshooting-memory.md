---
title: "자바 트러블슈팅 - 메모리 진단하기"
categories:
  - Operation
tags: 
toc: true
toc_label: "Contents"
toc_sticky: true
---

## 자바 트러블 슈팅 - 메모리 진단하기

### 12. 메모리 때문에 발생할 수 있는 문제들

#### 자바의 메모리 영역
* 실행 시 데이터가 저장되는 영역은 아래와 같이 나뉜다.
  * pc(program counter) 레지스터: 스레드별 보유, 각 스레드의 JVM 인스트럭션 주소 저장
  * JVM 스택: 스레드별 보유, 스레드가 생성되면서 동시에 생성되며 지역 변수와 부분 결과 및 메서드 호출과 리턴과 관련된 정보 보관
  * 힙(heap): 대부분의 데이터가 저장되는 일반적인 저장소, 모든 클래스의 인스턴스와 배열 할당, JVM이 시작될 때 생성되며 가비지 컬렉터에 의해 관리
  * 메서드 영역: 모든 JVM의 스레드를 공유하며 각 클래스의 구조 정보 저장, 런타임 상수 풀, 필드, 메서드 데이터, 메서드의 생성자의 코드, 클래스와 인터페이스 인스턴스의 초기화를 위한 특수 메서드들에 대한 정보
  * 런타임 상수 풀(Runtime Constant Pool): 메서드 영역에 할당
  * 네이티브 메서드 스택: 자바 언어 이외의 네이티브 언어를 호출할 경우 타 언어의 스택 정보 저장
* 힙 메모리 영역은 아래와 같이 구분된다.
  * Young: Eden, Survivor 0, Survivor 1
  * Tenured: Old
  * Permanent: Perm
  * G1의 경우 바둑판 모양으로 나뉘며 일부를 New 영역, 나머지를 Old 영역으로 정의해서 사용한다. New 영역이 꽉 차면 자신을 Old 영역으로 변환한다.

#### OutOfMemoryError는 언제 발생할까?
* 가비지 컬렉터가 새로운 객체를 생성할 공간을 더 이상 만들어주지 못하고 더 이상 힙 영역의 메모리가 증가될 수 없을 때
* 네이티브 라이브러리 코드에서 스왑(swap) 영역이 부족하여 더 이상 네이티브 할당을 할 수 없을 때(드물게 발생)

#### OutOfMemoryError 메세지의 의미
* `Exception in thread "main": java.lang.OutOfMemoryError: Java heap space`
  * 메모리 크기를 너무 적게 잡아 놓거나 지정하지 않은 경우
  * 오래된 객체들이 계속 참조되고 있어 GC가 되지 않는 경우: `static`을 잘못 사용하는 등
  * `finalize` 메서드를 개발자가 개발한 클래스에 구현해 놓은 경우
  * 스레드의 우선순위를 너무 높일 경우
  * 큰 덩어리의 객체가 여러 개 있을 경우
* `Exception in thread "main": java.lang.OutOfMemoryError: Metaspace`
  * 너무 많은 클래스가 해당 자바 프로세스에 로딩될 경우
* `Exception in thread "main": java.lang.OutOfMemoryError: Requested array size exceeds VM limit`
  * 배열의 크기가 힙 영역의 크기보다 더 크게 지정된 경우
* `Exception in thread "main": java.lang.OutOfMemoryError: request <size> bytes for <reason>. Out of swap space?`
  * 네이티브 힙 영역이 부족한 경우
  * OS의 Swap 영역까지도 부종한 경우 발생한다. 즉, 개발된 자바 애플리케이션에서 호출하는 네이티브 메서드에서 메모리를 반환하지 않거나 다른 애플리케이션에서 메모리를 반환하지 않는 경우 발생한다.
* `Exception in thread "main": java.lang.OutOfMemoryError: <reason> <stacktrace> (Native method)`
  * 네이티브 힙 영역에 메모리를 할당할 때 발생하는 메시지로 메모리 할당 오류가 JNI나 네이티브 코드에서 발생하는 경우

#### 메모리 릭의 세 종류
* 수평적 메모리 릭
  * 하나의 객체에서 매우 많은 객체를 참조하는 경우
* 수직적 메모리 릭
  * 각 객체들이 링크로 연결되었을 경우
* 대각성 형태의 메모리 릭
  * 객체들이 복합적으로 메모리를 점유하는 경우

#### OutOfMemoryError 이외의 메모리 문제는 없을까?
* 크래시가 발생하는 경우가 있다. 메모리 문제가 있을 때 종종 크래시가 발생하는데, 이 경우 서버의 자바 프로세스가 사라지게 된다. 보통 네이티브 힙에 메모리 할당이 실패하면 발생한다.
* 너무 잦은 (full) GC가 있다. GC 튜닝을 하기보다는 GC가 발생하지 않도록 하는 것이 선무다.
  * 임시 메모리의 사용 최소화, 객체의 재사용, 너무 많은 데이터를 한 번에 보여 주는 비즈니스 로직 제거 등
  * 모니터링 도구, `verbosegc` 옵션 사용, `jstat` 사용 등으로 GC가 얼마나 발생하는지 확인 가능하다.

### 13. 메모리 단면 잘라 놓기

#### 메모리 단면은 언제 자르나?
* 메모리 단면인 힙 덤프는 메모리가 부족해지는 현상이 지속해서 발생할 때와 `OutOfMemoryError`가 발생했을 때 생성해야 한다.
* 메모리가 부족한 걸 어떻게 알까?
  * `jstat`으로 확인
  * WAS 모니터링 콘솔
  * JMX 기반 모니터링 도구
  * APM
  * `verbosegc` 옵션으로 확인
* `jstat`를 사용해 Old 영역(O)의 메모리 사용량이 GC 이후에도 증가하는지 확인하거나 모니터링 도구 등을 사용해서 알 수 있다.
  * Old 영역의 메모리 사용량은 애플리케이션을 사용하고 있는 동안에는 계속 증가하는 것이 기본이며 Full GC가 발생한 이후 메모리 사용량이 증가해야만 메모리 릭이 발생하고 있다고 볼 수 있다.
  * 성능 프로파일링 도구를 사용하면 성능에 큰 영향을 미치므로 개발 서버에서 문제점을 확인하는 게 좋다.
  * 메모리 단면을 자르는 일은 덤프 파일을 생성하는 동안 서비스가 불가능한 상황이 되고, 덤프 생성 시 너무 많은 시간이 소요되며 큰 파일이 생성된다는 비용이 있다.

#### jmap으로 메모리 단면 생성하기
* `jmap`은 프로세스 id만 알고 있으면 메모리 단면을 생성할 수 있다. 
  * 다양한 옵션을 제공한다. (`-finalizerinfo`, `clstats`, `-histo` 등)
  * GC를 수행하려고 대기 중인 객체 출력, 각 객체의 타입별로 점유하고 있는 바이트의 크기, 가장 많은 메모리를 점유한 객체부터 데이터 출력 등

#### jmap의 -dump 옵션 사용하기
* 메모리 단면을 생성하는 일은 사용자가 덤프를 생성하는 동안 응답을 받지 못하게 하므로 사용자의 접근을 막고 사용해야 한다.
* `jmap -dump:[live,]format=b,file=<filename>`의 형식으로 사용한다.

#### 자동으로 힙 덤프 생성시키기
* `-XX:+HeapDumpOnOutOfMemoryError` 옵션을 사용하면 `OutOfMemoryError` 발생 시 힙 덤프를 자동으로 발생시킬 수 있다.
  * 경로 옵션도 지정 가능하다.
* `-XX:OnOutOfMemmoryError="명령어"`를 통해 해당 에러 발생 시 수행되는 명령어나 스크립트도 명시 가능하다.
* `jmap`이 동작하지 않는 경우 `gcore`를 사용한다.

### 14. 잘라 놓은 메모리 단면 분석하기

#### 메모리 단면을 분석하는 도구들
* MAT, IBM Heap Analyzer 등이 있다.
* MAT
  * 객체에 대한 설명, 객체 트리 목록 등을 제공한다.
  * Inspector 창에서는 선택된 객체의 주소, 패키지, 부모 클래스 등의 상세 정보를 제공한다.
  * Overview 탭에서는 메모리 단면 파일의 요약 정보, 원 그래프, 세부 정보, 보고서 메뉴 등을 제공한다.

### 15. 메모리 문제 Case Study

#### 시스템이 느리다고 항상 메모리 단면을 사용하는 것은 아니다
* 메모리 문제로 애플리케이션 응답이 느린 경우 다음이 원인일 수 있다.
  * 메모리 크기를 잡지 않거나 너무 작게 잡아 GC가 너무 자주 발생하는 경우
  * 임시 메모리를 많이 사용하여 GC가 자주 발생하는 경우
  * GC 때문에 문제가 발생하는 경우 `jstat` 명령을 사용하여ㅑ 원인을 파악할 수 있다. `-gccapacity`로 메모리 크기, `-gcutil`로 메모리 사용량을 파악한다.

#### 애플리케이션이 응답하지 않을 때도 메모리가 원인일 수 있다
* 애플리케이션이 응답하지 않을 때 메모리와 관련된 것 중 원인일 확률이 높은 것은 메모리 릭이다. 메모리 릭이 원인인 경우 아래의 절차를 따른다.
  * 현재 메모리 사용량을 확인해 본다. 지속해서 95% 이상인 경우 메모리 릭이 원인일 확률이 높다. (다만 full GC 후 메모리 사용량이 20~50% 정도로 떨어진다면 메모리 관련 문제가 아닐 확률이 높다.)
  * 지속해서 높은 경우 메모리 단면을 생성한다.
  * 도구를 이용해 어떤 객체가 죽지 않고 점유되고 있는지 확인해 본다. 

#### 정리
* 메모리 단면을 생성하는 동안 서비스가 원활하게 이루어지지 않기 때문에 `jstat` 같은 도구로 메모리 원인으로 발생하는 문제인지를 먼저 확인하는 게 좋다.
* `jstat`, 스레드 단면, WAS 로그, 메모리 단면 순