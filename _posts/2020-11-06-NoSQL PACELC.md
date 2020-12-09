---
title: "NoSQL PACELC"
permalink: /nosql/PACELC
categories: nosql PACELC
---

# NoSQL PACELC 이론

## ① Overview

NoSQL 의 특징 중 하나는 수평 확장성(Horizontal Scalability)이다. 수평적으로 확장되면 여러 노드로 구성된다. 이때 노드 끼리는 네트워크 통신을 하게 될텐데 현실적으로 100% 장애 없는 네트워크는 존재할 수 없다.

그런데 CAP 이론에서는 장애 상황에 대한 설명이 없다. 그래서 모든 케이스를 CAP 이론으로 설명할 수 없다는 의견이 나오고 그 대안으로 사용되는 이론이다. PACELC 는 장애 상황과 정상 상황을 구분하여 케이스를 분리하는 것이 CAP 과 크게 다른 특징이다.

---

## ② PACELC

![PACELC 이론1.jpg](http://itwiki.kr/images/thumb/5/53/PACELC_%EC%9D%B4%EB%A1%A01.jpg/400px-PACELC_%EC%9D%B4%EB%A1%A01.jpg)

* PACELC 는 장애 상황과 정상 상황, 두 가지 케이스로 상황을 나눈다. 그리고 앞 글자를 따 `PAC` `ELC ` 라는 이론으로 불린다.

#### PAC

* 약자
  * P : Network Partition, 네트워크 간 파티션으로 나뉘어진 상황. 즉, 장애 상황을 말한다.
  * A : Availability
  * C : Consistency
* 설명
  * 장애 상황(Network Partition)에서는 가용성(Availability) 과 일관성(Consistency) 는 trade-off 관계이다.
* 예를 들어
  * 장애 상황에서 Availability 를 중요하게 여기는 시스템은 Consistency 가 낮다.
  * 장애 상황에서 Consistency 를 중요하게 여기는 시스템은 Availability 가 낮다.

#### ELC

* 약자
  * E : Else, PAC 에서 P(Network Partition) 가 아닌 경우. 즉, 정상적인 상황을 말한다.
  * L : Latency
  * C : Consistency
* 설명
  * 정상적인 상황(E)에서는 지연 시간(Latency) 와 일관성(Consistency) 는 trade-off 관계이다.
* 예를 들어
  * 정상적인 상황에서 Consistency 를 중요하게 여기는 시스템은 Latency 가 길어진다.
  * 정상적인 상황에서 Latency 를 중요하게 여기는 시스템은 Consistency 가 잘 지켜지지 않는다.

---

## ③ PA/EL, PA/EC, PC/EL, PC/EC

* PACELC 이론에서는 시스템 종류를 네 가지로 분류할 수 있다.
* [wikipedia Database PACELC ratings 참고](https://en.wikipedia.org/wiki/PACELC_theorem)

#### PA/EL

* Cassandra, DynamoDB
  * Cassandra, DynamoDB 둘 다 설정을 통해 Latency 와 Consistency 의 trade-off 를 선택할 수 있다.
* 장애 상황 → PA
  * 가능한 노드에만 데이터를 반영하고
  * 장애 노드가 복구되면 그 노드에도 반영한다. (Eventual Consistency)
* 정상 상황 → EL
  * 정상 상황일때도 짧은 Latency 를 위해 모든 노드에 데이터를 즉시 반영하지 않는다. (Eventual Consistency)

#### PC/EL

* HBase
* 장애 상황 → PC
  * 일관성(Consistency) 를 위해 가용성(Availability) 를 희생한다.
* 정상 상황 → EL
  * 일관성(Consistency) 를 위해 긴 지연 시간(Latency) 도 감수한다.

#### PA/EC

* MongoDB
* 장애 상황 → PA
  * 일단 정상적인 노드에만 데이터를 반영하고, 장애 노드는 복구되면 반영한다.
  * 일관성을 포기한다.
* 정상 상황 → EC
  * 모든 노드에서 동일한 데이터를 볼 수 있다.
  * 강력한 일관성을 제공한다.

#### PC/EL

* Couchbase (PC/EL + EC), PNUTS
* 장애 상황 → PC
  * 일관성을 위해 가용성을 희생한다. 노드간 네트워크 장애가 발생한 경우 Consistency 를 지키기 위해 장애 시간만큼 Availability 를 희생시킨다.
* 정상 상황 → EL
  * 평소에는 짧은 Latency 를 위해 Consistency 를 희생한다.

---

## ④ CAP, PACELC 이론으로 RDBMS 를 분류하지 말자

* 애초에 CAP, PACELC 이론은 분산 시스템으로 설계되는 NoSQL 을 표현? 분류? 하기 위한 이론이다. 하지만 RDBMS 는 초기 설계부터 분산 시스템으로 고려되지 않기 때문에 CAP, PACELC 이론으로 분류하는 것은 적합하지 않다.

---

## 참고

* [CAP Theorem, 오해와 진실](http://eincs.com/2013/07/misleading-and-truth-of-cap-theorem/)
* [wikipedia - PACELC theorem](https://en.wikipedia.org/wiki/PACELC_theorem)
* [http://itwiki.kr/w/PACELC_%EC%9D%B4%EB%A1%A0](http://itwiki.kr/w/PACELC_%EC%9D%B4%EB%A1%A0)

