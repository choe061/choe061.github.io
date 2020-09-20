---
title: "Spring Cloud Stream Dynamic Routing"
permalink: /spring/spring-cloud-stream/3
categories: spring spring-cloud-stream
---

# spring-cloud-stream(3) Dynamic Routing

## Dynamic Routing

* application 로직에 따라 동적 라우팅이 필요한 경우가 많을 것 같다. 하나의 destination 에 있는 메시지를 여러 consumers 들 중에서 동적으로 결정할 수도 있고, producer 가 여러 destination 중 동적으로 하나의 destination 을 결정할 수도 있다.
* 비즈니스 로직에 따라 Process 가 달라지는 경우 Dynamic Routing 을 활용하면 유용할 것 같다.

---

## Route TO Consumer

* consumer 를 런타임에 결정

---

## Route FROM Consumer

* produce destination 을 런타임에 결정

###### 구현 방법

* (Deprecated) ~~BinderAwareChannelResolver~~
* `spring.cloud.stream.sendto.destination` Header

```java
@Configuration
public class DynamicDestinationProcessor {
    @Bean
    public Function<InputDTO, Message<? extends OutputDTO>> destinationAsPayload() {
        return input -> {
            var result = process(input);
            if (isSuccess(result)) {
                var aOutputDTO = toAOutputDTO(result);
                return MessageBuilder.withPayload(aOutputDTO)
                				 .setHeader("spring.cloud.stream.sendto.destination", "aDest")
                				 .build();
            }
            
            var bOutputDTO = toBOutputDTO(result);
            return MessageBuilder.withPayload(bOutputDTO)
                				 .setHeader("spring.cloud.stream.sendto.destination", "bDest")
                				 .build();
        };
    }
}
```

---

## vs Producer Partitioning

---

## 참고

* [event routing](https://cloud.spring.io/spring-cloud-static/spring-cloud-stream/current/reference/html/spring-cloud-stream.html#_event_routing)

