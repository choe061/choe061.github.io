---
title: "Spring Cloud Stream Error Handling"
permalink: /spring/spring-cloud-stream/4
categories: spring spring-cloud-stream
---

# spring-cloud-stream(4) Error Handling (with RabbitMQ)

## Error Handling

* spring-cloud-stream 에서 에러를 처리하기 위한 메커니즘을 제공한다. 하지만 각 binder 별로 세부적인 기능 차이가 있기 때문에 binder 가 변경된다면, 에러 처리 부분은 확인이 필요하다. 여기서는 RabbitMQ 를 기준으로 정리해보려고 한다.
  * Handler(Function) 이 예외를 던지면, binder 로 예외를 전파한다. 그러면 binder 는 messaging system 으로 에러를 전달한다. 세부적인 기능은 messaging system 의 기능에 따라 다르지만 메시지를 `drop`, `re-queue`, `DLQ` 로 전송할 수 있다. Rabbit 과 Kafka 같은 binder 는 이 개념들을 모두 지원한다. 하지만 다른 binder 의 경우 지원하지 않을 수도 있다. 그래서 error-handling 옵션은 각각의 binder documentation 을 확인해야 한다.
* spring cloud stream 은 Annotation Handler 과 명령형 Function, Reactive Function 방법이 있다. 명령형 Function 과 Handler 는 Spring Retry 를 사용하고, Reactive Function 은 reactive API 의 retryBackoff 를 사용한다.

---

#### Drop Failed Messages

* 추가적인 system-level 의 설정을 추가하지 않으면 messaging system 은 실패한 메시지를 drop 한다. 메시지가 유실되지 않으려면 복구 메커니즘이 필요!

---

#### DLQ - Dead Letter Queue

* Failed Messages 를 특정한 destination 으로 다시 전송하는 것.

* DLQ 의 메시지를 재처리, 모니터링, 보정(수정)

* RabbitMQ 에서 DLQ 사용 방법

  ```YAML
  spring:
  	cloud:
  		stream:
  			bindings:
          		message-in-0:
            			binder: rabbit
            			destination: message
            			group: messageConsumers
  			rabbit:
  				bindings:
  					message-in-0:
  						consumer:
  							auto-bind-dlq: true
  ```

  * `auto-bind-dlq` 를 true 로 설정하면 `message.messageConsumers.dlq` 로 RabbitMQ queue 가 추가된다. DLQ 설정이 완료되면, 실패한 메시지는 기존 데이터와 함께 오류 메시지를 포함하여 DLQ 로 전송된다.

    * Message 의 `x-exception-stacktrace` Header 에 error message 가 있다.

      ```powershell
      delivery_mode:	2
      headers:
      x-original-exchange:
      x-exception-message:	has an error
      x-original-routingKey:	input.myGroup
      x-exception-stacktrace:	org.springframework.messaging.MessageHandlingException: nested exception is
            org.springframework.messaging.MessagingException: has an error, failedMessage=GenericMessage [payload=byte[15],
            headers={amqp_receivedDeliveryMode=NON_PERSISTENT, amqp_receivedRoutingKey=input.hello, amqp_deliveryTag=1,
            deliveryAttempt=3, amqp_consumerQueue=input.hello, amqp_redelivered=false, id=a15231e6-3f80-677b-5ad7-d4b1e61e486e,
            amqp_consumerTag=amq.ctag-skBFapilvtZhDsn0k3ZmQg, contentType=application/json, timestamp=1522327846136}]
            at org.spring...integ...han...MethodInvokingMessageProcessor.processMessage(MethodInvokingMessageProcessor.java:107)
            at. . . . .
      Payload {"name”:"Bob"}
      ```

---

#### Re-queue Failed Messages

- 레빗이나 카프카 등.. 메시지 재처리를 위해 내부적으로 RetryTemplate 을 이용한다.

- 디폴트 설정을 사용하면 최대 3 번 requeue 한다. max-attempts 를 넘은 메시지는 Drop.

- max-attempt property 를 1 로 설정하면, 내부적인 재처리 로직은 disable 된다. 1이 아닌 수를 넣게되면 최대 n 번 만큼 재처리하는 loop 를 돌게 된다.

  - 두 가지 properties

  ```yaml
  spring:
  	cloud:
  		stream:
  			bindings:
  				message-in-0:
  					consumer:
  						max-attempts: 1
  			rabbit:
  				bindings:
  					message-in-0:
  						consumer:
  							requeue-rejected: true
  ```

#### Retry Template and retryBackoff

* retry configuration
* 두 가지 다른 메커니즘
  * imperative → RetryTemplate → Spring Retry
  * reactive → retryBackoff

###### maxAttempts

* 메시지 재시도 횟수
* Default : 3

###### backOffInitialInterval

* retry 간격
* Default : 1000ms

###### backOffMaxInterval

* backOffInitialInterval 옵션의 최대 값
* Default : 10000ms

###### backOffMultiplier

* The backoff multiplier ?
* Default : 2.0

###### defaultRetryable

* `retryableExceptions` 에 정의되지 않은 exception 이 listener 에서 발생하면 retry 할지 결정하는 옵션
* Default : true

###### retryableExceptions

* map 으로 정의
  * key - Throwable class name
  * value - boolean
* 특정 exception (and subclasses) 에 대한 retry 여부를 결정
* `spring.cloud.stream.bindings.input.consumer.retryable-exceptions.java.lang.IllegalStateException=false`
* Default : empty

#### Custom Retry

- 몇 가지 Properties 를 제공하지만, 복잡한 요구사항을 구현하기 위해 커스텀이 필요한 경우 `RetryTemplate` 을 직접 구현하여 제공하는 방법도 있다. Application configuration 에서 RetryTemplate 을 bean 으로 설정하면 된다.

- 기존의 RetryTemplate 과 conflict 를 피하기 위해 binder 에 사용하려는 RetryTemplate 에 `@Bean` 이 아닌 `@StreamRetryTemplate` 을 붙여야 한다. (내부적으로 qualified @Bean)

  ```java
  @StreamRetryTemplate
  public RetryTemplate myRetryTemplate() {
  	return new RetryTemplate();
  }
  ```

  <img src="/Users/a1101083/Library/Application Support/typora-user-images/image-20200927113053754.png" alt="image-20200927113053754" style="zoom:50%;" />

- binding 별로 세부적으로 Retry 정책을 달리하고 싶다면, property 를 설정할 수도 있다.

  - `spring.cloud.stream.bindings.<foo>.consumer.retry-template-name=<your-retry-template-bean-name>`

---

## 참고

* [spring-cloud-stream error handling](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.6.RELEASE/reference/html/spring-cloud-stream.html#spring-cloud-stream-overview-error-handling)

