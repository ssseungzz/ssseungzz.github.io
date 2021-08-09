---
title: "API Composition이란?"
categories:
  - Develop
tags:
  - "design-pattern"
toc: true
toc_label: "Contents"
toc_sticky: true
---

## API Composition이란?

* 어쩌다 나온 걸까?
  * [마이크로서비스 아키텍처](https://ko.wikipedia.org/wiki/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4)의 적용(+ 서비스마다 데이터베이스 두기) 이후, 여러 서비스로부터 데이터를 조인하는 쿼리를 구현하는 것이 쉽지 않아졌다.
  * 또 서비스 사이의 복잡도가 증가하게 되면 서비스 간 통신에 필요한 API의 호출의 빈도도 증가한다. ➡️ 그만큼 오버헤드가 증가한다.
  * Rest API도 기능별로 나누어져 클라이언트가 호출해야 하는 API도 늘어나고, 자연스레 호출의 빈도도 증가하게 된다.
* 어떻게 마이크로서비스 아키텍처에서 쿼리를 조금이라도 더 쉽게 구현할 수 있을까? 또, 어떻게 해야 클라이언트가 호출해야 하는 API의 개수를 줄일 수 있을까?
  * 하나의 요청으로 여러 서비스에 요청하고 응답을 모아 다시 하나의 응답으로 받을 수 있다면 이를 이룰 수 있다.
* 이런 목적을 달성하기 위해 API Composition(*합성*)을 이용하면 된다!

### 그래서 어떻게 합성하는 건데?

![](https://microservices.io/i/data/ApiBasedQueryBigPicture.png)

* API Composer를 정의함으로써 쿼리를 구현한다.
  * composer는 데이터를 가지고 결과의 인메모리 조인을 수행하는 서비스를 수행한다.
  * 즉, 여러 서비스의 API 호출 결과를 조합하여 응답하는 방식이다.
  * 대량의 데이터나 복잡한 데이터를 응답하는 경우에는 적절하지 않다. 
* 예로 API Gateway가 이런 역할을 수행해 준다.
  * API의 결과를 합성해 주는 레이어는 API Gateway 앞, 내부, 뒤에 위치할 수 있는데 각각의 위치에 따른 장단이 존재한다.
    * 앞: Gateway를 건드리지는 않지만 합성 레이어가 엔드 포인트가 되면서 트래픽을 부담하게 된다. 둘 사이의 latency는 최소화되어야 한다.
    * 내부: Gateway 내부에서 동작하기 때문에 network hop이 존재하지 않지만, 구현이 복잡해지고 성능에 영향을 줄 수 있다. 로직 변경 시 재시작이 필요할 수 있다.
    * 뒤: 합성 층이 API Provider가 된다. 합성 층을 위한 엔드 포인트를 API Provider가 제공해야 할 수 있다.
* 설계 시에는 API 설계 → 동기/비동기 설계 →  트랜잭션 설계 → 데이터 조회 설계 순으로 진행한다.
  * 마이크로서비스마다 DB가 분리되어 물리적 조인이 불가능한 경우를 염두에 두어야 한다.

#### References

* https://microservices.io/patterns/data/api-composition.html