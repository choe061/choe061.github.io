---
title: "Spring Cloud Stream 설정 & 구현"
permalink: /spring/spring-cloud-stream/2
categories: spring spring-cloud-stream
---

# spring-cloud-stream(2) Configuration & Implementation

## message-middleware 및 구현 방법 선택

* RabbitMQ
* Functional Binding

---

## Configuration

#### Binder Configuration

* /resources/`META-INF/spring.binders` 파일에 binder 구현체 정의

  ```binders
  rabbit:\
  org.springframework.cloud.stream.binder.rabbit.config.RabbitMessageChannelBinderConfiguration
  
  local_rabbit:\
  org.springframework.cloud.stream.binder.rabbit.config.RabbitMessageChannelBinderConfiguration
  ```

  * .properties, .yml 에서 binder 의 이름을 local_rabbit 으로 사용하려면 spring.binders 에서 local_rabbit 으로 정의해야 한다.

  * spring-cloud-starter-stream-rabbit 을 디펜던시로 추가하면 spring.binders 파일을 정의하지 않아도 디폴트로 `rabbit` 이름의 binder 를 사용할 수 있다.

    * ```groovy
      implementation 'org.springframework.cloud:spring-cloud-starter-stream-rabbit'
      ```

    * spring-cloud-stream-binder-rabbit 모듈의 `META-INF/spring.binders` 파일에 `rabbit` 이 정의되어 있다.

#### YAML Configuration

```yaml
spring:
  cloud:
    stream:
      function:
      	definition: message
      bindings:
        message-in-0:
          destination: message
          binder: local_rabbit
          group: messageConsumers
          consumer:
            concurrency: 8
        message-out-0:
          destination: push
          binder: local_rabbit
      binders:
        local_rabbit:
          type: rabbit
          environment:
            spring:
              rabbitmq:
                host: <host>
                port: 5672
                username: <username>
                password: <password>
                virtual-host: /
```

* function.definition
  * 정의한 function(binding) 을 나열
  * 두 가지 splitter
    * `;` 로 파이프라인 분리
    * `|` 로 파이프라인 연결
  * ex) `definition: order|pay|message;stat`
    - order → pay → message 가 하나의 파이프라인
    - stat 이 하나의 파이프라인
* bindings
  * functional bindings naming conversions

    * consumer → `<function name>-in-<index number>`
    * producer → `<function name>-out-<index number>`

  * destination

    * rabbitmq 의 Exchange

  * binder

    * spring.binders 파일에 정의한 binder 지정

  * group

    * consumer property
    * multiple instance 의 경우 group 속성을 지정하지 않으면, 하나의 메시지를 여러 instance 에서 consume 할 수 있다. 하나의 메시지는 한 번만 처리되어야 한다면 group name 을 동일하게 지정해야 한다.

  * [consumer properties](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.6.RELEASE/reference/html/spring-cloud-stream.html#_consumer_properties)

    * `spring.cloud.stream.bindings.<binding-name>.consumer.`
      * spring cloud stream 에서 공통으로 지원하는 consumer properties
    * `spring.cloud.stream.rabbit.bindings.<channel-name>.consumer.`
      * **⚠️ stream 의 consumer properties 와 다르다. 헷갈리지 않도록 주의!!!**
      * binder 가 rabbitmq 일때 사용 가능한 consumer properties
      * [rabbit consumer properties](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream-binder-rabbit/3.0.6.RELEASE/reference/html/spring-cloud-stream-binder-rabbit.html#_rabbitmq_consumer_properties)

  * [producer properties](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/3.0.6.RELEASE/reference/html/spring-cloud-stream.html#_producer_properties)
  * consumer 와 마찬가지로 spring-cloud-stream 에서 공통으로 지원하는 properties 가 있고, [rabbit producer properties](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream-binder-rabbit/3.0.6.RELEASE/reference/html/spring-cloud-stream-binder-rabbit.html#rabbit-prod-props) 가 있다.

#### 상용 환경에서 필요한 설정

###### Scaling Up

* multiple application 으로 운영될 때, 데이터가 여러 consumer 에 골고루 나뉘는 것도 자원을 효율적으로 사용하는 것이므로 중요하다. 그렇게 하기 위해서 두 가지 properties 를 사용해야 한다.

* properties
  * `spring.cloud.stream.instanceCount`
    * 운영중인 Application 개수
    * default : 1
    * partitioning 및 kafka 를 사용하면 설정해야 한다.
  * `spring.cloud.stream.instanceIndex`
    * 현재 Application 의 index
    * Cloud Foudary 에서 자동으로 Application 의 instance index 를 설정해줌
    * partitioning 및 kafka 를 사용하면 설정해야 한다.

* Example) MyLoggerServiceApplication 을 2개의 instance 로 배포한다면
  * spring.cloud.stream.instanceCount → 2
  * spring.cloud.stream.instanceIndex → 각각 0 과 1
* Auto-scaling 환경에서는 어떻게 예측하는가?
  * https://github.com/spring-cloud/spring-cloud-stream/issues/1342

###### partitioning

* domain event 는 messages 를 어떤 key 를 기준으로 분할할 수도 있다.
* Example) Log Message 의 첫번째 문자가 partition key 가 된다. 첫번째 문자를 기준으로 두 파티션으로 그룹화하여 첫번째 문자가 A~M 파티션과 N~Z 파티션으로 나뉜다.
  * 두 가지 properties
    * `spring.cloud.stream.bindings.output.producer.partitionKeyExpression`
      * payloads 를 분할하는 방법에 대한 표현식
      * 표현식이 복잡하여 한 줄로 작성하기 어려운 경우
        * `spring.cloud.stream.bindings.output.producer.partitionKeyExtractorClass` property 를 사용하여 custom partition strategy 를 사용할 수 있다.
    * `spring.cloud.stream.bindings.output.producer.partitionCount`
      * 파티션 그룹의 번호

###### Health Indicator

* Microservice 환경에서 서비스의 down 과 start falling 을 감지해야 한다. 이를 위해 Spring Cloud Stream 은 binders 에 대한 Health Indicator 를 활성화시키기 위해 `management.health.binders.enabled` property 를 제공한다.

###### [multiple binders](https://docs.spring.io/spring-cloud-stream/docs/Brooklyn.RELEASE/reference/html/_binders.html#multiple-binders)

* spring.cloud.stream.defaultBinder
* multiple binders 를 사용한다면 default binder 를 지정해야 한다.

---

## Implementation

#### Function<T, R>

* consume 한 메시지를 가공하고 다른 작업을 위해 다시 produce 하는 경우

```java
@Configuration
@RequiredArgsConstructor
public class MessageProcessor {
    private final SendMessageService service;
    
    @Bean
    public Function<MessageInputDTO, MessageOutputDTO> message() {
        return dto -> process(dto);
    }
    
    private MessageOutputDTO process(MessageInputDTO input) {
        var message = toEntity(input);
        var result = service.send(message);
        return toOutput(result);
    }
    
    // toEntity, toOutput method
}
```

#### Consumer\<T\>

* 메시지를 consume 하여 처리하고 끝내는 경우

```java
@Configuration
@RequiredArgsConstructor
public class MessageConsumer {
    private final SendMessageService service;
    
    @Bean
    public Consumer<MessageInputDTO> message() {
        return dto -> process(dto);
    }
    
    private void process(MessageInputDTO input) {
        var message = toEntity(input);
        service.send(message);
    }
    
    // toEntity method
}
```

#### Supplier\<R\>

* 메시지를 생산하여 다른 작업을 위해 produce 하는 경우

```java
@Configuration
@RequiredArgsConstructor
public class MessageProcessor {
    private final GenerateMessageService service;
    
    @Bean
    public Supplier<MessageOutputDTO> message() {
        return dto -> process(dto);
    }
    
    private MessageOutputDTO process() {
        var message = service.subscribe();
        return toOutput(message);
    }
    
    // toOutput method
}
```

* 데이터를 어디선가 생산해서 넣어주어야 Supplier 가 동작을 하는데, 이 경우 Queue 을 이용하면 된다.

```java
public class GenerateMessageService {
    private final BlockingQueue<MyMessage> messageQueue = new LinkedBlockingQueue<>();
    
    public void send(MyMessage message) {
        var result = messageQueue.offer(message);
        if (!result) {
            log.error("message offer error. message: {}", message, e);
            throw new SendMyMessageExcpetion();
        }
    }
    
    public MyMessage subscribe() {
        try {
        	return messageQueue.take();
        } catch (InterruptedException e) {
            log.error("message take error. ", e);
            throw new MyMessageSubscriptionException();
        }
    }
}
```

* API 호출을 통해 send(MyMessage message) method 를 호출하여 메시지를 BlockingQueue 에 넣으면 Supplier 에서는 BlockingQueue 의 데이터를 take() method 를 통해 가져와 최종적으로 produce 한다.

* `BlockingQueue<Long> q = new LinkedBlockingQueue<>()`
  * BlockingQueue 의 take(), poll(), poll(long timeout, TimeUnit unit)
    * take, poll 모두 데이터를 조회하지만
    * poll 은 조회할 수 있는 element 가 없으면 timeout 만큼만 대기하다가 null 을 리턴한다.
    * take 는 조회할 수 있는 element 가 있을 때까지 대기한다.

---

## 참고

* [spring-cloud-stream document](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/current/reference/html/spring-cloud-stream.html)
* [Java BlockingQueue - baeldung](https://www.baeldung.com/java-blocking-queue)