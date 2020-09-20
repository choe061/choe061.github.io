---
title: "publish/subscribe vs produce/consume"
permalink: /dev/publish-subscribe-vs-produce-consume
categories: publish subscribe produce consume
---

# Pub/Sub vs Produce/Consume

- 메시징 시스템 관련 용어. 비슷하지만, 다른 개념

- [wikipedia]([https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern](https://en.wikipedia.org/wiki/Publish–subscribe_pattern)) 참고

  > Publish–subscribe is a sibling of the [message queue](https://en.wikipedia.org/wiki/Message_queue) paradigm, and is typically one part of a larger [message-oriented middleware](https://en.wikipedia.org/wiki/Message-oriented_middleware) system. 

  * Message Queue 패러다임의 형제 개념이고, 큰 관점에서 message-oriented middleware 시스템 중 하나.

---

## Publish/Subscribe

- 메시지가 multiple Receivers 에 분배되는 메시징 패턴

---

## Produce/Consume

- 일종의 버퍼 확장 개념
  - 데이터 생산 속도가 처리 속도를 초과하면 throughput(처리량) 이 떨어지고, 처리하는 쪽에서 문제가 발생할 수 있다. 이 문제를 해결하기 위해 버퍼의 개념을 사용하여 MessageQueue 를 사이에 두어 Producer(생산자)와 Consumer(소비자)를 나누는 개념.

---

## 공통점

* 노드 구성 관점에서 1:1 부터 N:M 까지 노드를 구성할 수 있다.
* 시스템간 결합도 ↓

## 차이점

* 하나의 메시지 관점에서 관여하는 노드의 관계가 다르다.
  * Publisher/Subscriber 는 메시지를 발행하고 구독하는 노드의 관계가 1:1 또는 1:M
  * Producer/Consumer 는 메시지를 생성하고 소비하는 노드의 관계가 오직 1:1

---

## 참고

* [https://stackoverflow.com/questions/42471870/publish-subscribe-vs-producer-consumer](https://stackoverflow.com/questions/42471870/publish-subscribe-vs-producer-consumer)

