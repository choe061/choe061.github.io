---
title: "Spring Cloud Stream Overview"
permalink: /spring/spring-cloud-stream/1
categories: spring spring-cloud-stream
---

# spring-cloud-stream(1) Overview

## 개념 & 기본

* spring cloud stream 은 Spring Boot, Spring Integration 으로 만들어진 event-driven or message-driven 마이크로서비스 구축을 도와주는 Framework 이다.
* Endpoints 간 통신은 RabbitMQ 또는 Apache Kafka 같은 message-middleware 기반으로 이루어진다. 서비스들은 자신의 domain event 를 endpoints (또는 channels) 에 publish 하여 통신한다.

---

## 핵심 개념

#### 구성도

![SCSt-overview.png](https://github.com/spring-cloud/spring-cloud-stream/blob/master/docs/src/main/asciidoc/images/SCSt-overview.png?raw=true)

#### Destination Binder

* RabbitMQ or Kafka 같은 messaging-middleware 와 연결하기 위한 컴포넌트
* spring-cloud-stream 의 binder 덕분에 많은 보일러플레이트 코드를 줄일 수 있다.
* 외부 시스템 연결, 메시지 라우팅, 데이터 타입 변환, 사용자 코드 호출

#### Binding

* 외부 메시징 시스템과 Application 의 Producer, Consumer 를 연결해주는 브릿지
* spring boot application 에서 messaging-middleware 와 직접 통신하지 않고, binding 을 통해 통신한다.

#### Message

* Producer, Consumer 가 destination binder 와 통신하기 위해 사용하는 표준 데이터 구조

---

## 통신 원리

* 지정된 메시지들은 destination 에 Pub-Sub messaging pattern 으로 전달된다.

  > #### 💡destination
  >
  > * 해당 channel 을 메시지 시스템 topic 과 연결해주는 설정.
  >
  > * RabbitMQ 에서는 Exchange 에 해당.

* Publisher 는 메시지를 Topic(각각의 이름으로 식별 가능)으로 분류한다.
* Subscriber 는 한 개 이상의 Topic 을 구독할 수 있다. middleware 는 메시지를 필터링해서 해당 Topic 을 구독하는 Subscriber 들에게 전달한다.

---

## 구현 방식

1. (Legacy) Annotation Binding
2. Functional Binding (spring cloud stream version 3.0.0 부터)
   - java.util.function package 의 functional interface 를 사용
     - Function\<T, R\>
     - Consumer\<T\>
     - Supplier\<R\>
3. Functional Binding + Reactive

> 👨🏻‍💻 spring-cloud-stream document 를 보면 annotation 방식은 legacy 라고 한다. 우리는 2번과 3번 중 하나를 선택할 수 있을 것 같다. 단순 Functional Binding 만 사용하는 것보다 Reactive 를 함께 사용하는 것이 몇 가지(지금까지 파악하기로는 한가지?) **편리한** 기능을 더 지원해주는데, 사용하지 않아도 다른 방법으로 구현 가능하므로 굳이 Reactive 를 사용하지 않아도 될 것 같다.
>
> * input, output 객체에 Tuple 같은 객체를 사용하여 multiple input/output 을 구현할 수 있다.
> * 하지만, functional 방식에서도 Message<?> 객체의 Header 를 수정하여 destination 을 변경하는 방식으로 multiple input/output 을 구현할 수 있다.

---

## 편리한 기능 & 장점

> Spring 환경에서 RabbitMQ 를 직접 사용했을 때 보다 편리했던 점.

* MQ 연동
  * Java Code 에서 RabbitMQ(binder) 관련 설정이 모두 없어진다. (spring-cloud-stream 의 Annotation binding 방식을 사용한다면 설정이 남지만,) Functional 방식을 사용한다면 RabbitMQ(binder) 연동 코드를 Java Code 에서 없앨 수 있다. 
    * `.properties` or `.yml ` 설정 파일에 모두 명시
* Error Handling, Retry
  * spring-cloud-stream 을 사용해도 어떤 binder 를 사용하냐에 따라 세부적으로 다르지만, RabbitMQ 기준으로 몇 가지 설정만으로 Error Handling 과 Retry, DLQ 사용이 매우 쉬운것 같다.

---

## 참고

* [Spring Cloud Stream Reference Documentation](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.6.RELEASE/reference/html/)
* [Introduction to Spring Cloud Stream - Baeldung](https://www.baeldung.com/spring-cloud-stream)

