---
title: "스프링 @Async"
permalink: /spring/async
categories: async
---

# Spring @Async

public method 에 @Async 를 붙여 비동기 동작을 유발시킬 수 있다. 접근제한자는 반드시 public 이어야 하며 같은 클래스 내에서 @Async method 호출은 비동기로 동작하지 않는다. 그 이유는 어노테이션 동작 방식이 AOP Proxy 기반이기 때문이다.

#### [spring async-example project](https://github.com/choe061/spring-example/tree/master/async-example)

아래 설명에 대한 예제 코드

## return type

* void
* CompletableFuture\<T\>

###### with CompletableFuture (상위 인터페이스 Future 도 가능)

리턴 타입을 받기 위해서는 CompletableFuture 를 리턴 타입으로 지정하여 받을 수 있다. 비동기로 실행된 코드의 결과 값을 받아오거나, CompletableFuture 의 기능을 활용하여 여러 비동기 코드의 결과 값을 취합할 수도 있다.

## Thread pool

#### 디폴트 Thread

아무런 설정을 하지 않는다면, 디폴트 구현체 SimpleAsyncTaskExecutor 를 사용한다. [docs.spring.io](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/SimpleAsyncTaskExecutor.html)의 NOTE 부분을 보면 **This implementation does not reuse threads!** 라고 나와 있다. Pool 을 두고 쓰레드를 재사용하는 것이 아닌 매번 생성하여 실행한다는 것이다.

#### Thread Pool 직접 지정

1. `@EnableAsync` 추가 → 이 설정은 Thread Pool 설정과 상관없이 @Async 를 사용하려면 추가해야 한다.
2. 새로운 Thread Pool Bean 등록

###### 1. Java configuration code

```java
@Bean
public Executor asyncThreadTaskExecutor() {
    ThreadPoolTaskExecutor threadPoolTaskExecutor 
        = new ThreadPoolTaskExecutor();
    threadPoolTaskExecutor.setCorePoolSize(8);
    threadPoolTaskExecutor.setMaxPoolSize(8);
    threadPoolTaskExecutor.setThreadNamePrefix("AsyncEmailTask-");
    return threadPoolTaskExecutor;
}
```

###### 2. application.yaml configuration (With Spring Boot 2.0 이상)

Spring Boot 2.0 부터는 yaml configuration 방식의 설정도 지원한다. 하지만, 이 경우 하나의 task execution 만 정의할 수 있는 것 같다.

```yaml
spring:
  task:
    execution:
      thread-name-prefix: AsyncEmailTask-
      pool:
        core-size: 8
        max-size: 8
```

###### Task 실행 결과

동시에 비동기 메소드를 여러번 호출하고, Async Method 에서 로그를 남겨보면 지정했던 prefix 가 붙은 Thread name 을 확인할 수 있다.

노란색은 요청으로 인해 톰캣의 Thread Pool 에서 가져온 쓰레드이다. 아래 빨간색 박스의 로그는 Async 메소드에서 찍은 로그이다. asyncThreadTaskExecutor Bean 에서 정의한 Prefix 가 붙어있다.

![async-log](https://raw.githubusercontent.com/choe061/spring-example/master/async-example/src/main/resources/threads-log.png)

###### pool 관련 속성

execution 의 pool 설정 값은 ThreadPoolExecutor 클래스를 참고하면된다. core-size 의 디폴트 값이 이상하게도 java 설정 방식과 yaml 설정 방식의 디폴트 값이 다르다.(..?)

* pool.core-size : idle 상태에서 pool 이 유지해야 하는 스레드 수
  * java config 방식 default 1
  * spring boot yaml 방식 default 8
* pool.max-size : pool 에 생성되는 최대 스레드 수
  * default `Integer.MAX_VALUE`

## Error Handling

* 참고 : [spring-async baeldung](https://www.baeldung.com/spring-async#exception-handling)

Async Method 를 사용하는 경우 에러 핸들링이 중요할 수 있다. 리턴 타입이 CompletableFuture(or Future) 인 경우는 결과에 대한 핸들링이 가능하지만, **리턴 타입이 void 인 경우** 별도의 처리 없이는 예외가 async 메서드를 호출한 thread 에 전달되지 않는다. 결국 exception 을 handling 하기 위한 async exception handler 를 구현해야 한다. 

@EnableAsync 어노테이션을 따라가다보면, Default 로 SimpleAsyncUncaughtExceptionHandler 를 사용하고 있다.

#### AsyncUncaughtExceptionHandler

AsyncUncaughtExceptionHandler interface 를 상속/구현하면된다. handleUncaughtException() 메서드가 있는데, async exception 이 발생하면 이 메서드가 실행된다. 

#### SimpleAsyncUncaughtExceptionHandler

Spring 4.1 부터 나온 위 인터페이스 AsyncUncaughtExceptionHandler 의 default 구현체이다. 하지만 운영 서버에서 예외 발생 시 어떠한 후처리를 하거나 각각의 프로젝트에서 사용하는 로그 패턴으로 남기는 방식으로 직접 구현하는 것도 좋을 것 같다.

아래는 예제에서 member number 가 5인 경우 강제로 throw exception 을 하는 코드를 작성하여 만든 에러이다. 기본적으로 SimpleAsyncUncaughtExceptionHandler 가 처리한다.

![simple-exception-handler-long](https://raw.githubusercontent.com/choe061/spring-example/master/async-example/src/main/resources/simpleAsyncUncaughtExceptionHandler-log.png)

#### AsyncConfigurer

Configuration 클래스에 AsyncConfigurer interface 를 상속 구현 받은 후 위에서 구현한 구현체(or 디폴트 구현체) 를 넣어준다.

###### 커스텀 ExceptionHandler 구현 방법

1. implements AsyncConfigurer
2. return type AsyncUnCaughtExceptionHandler, getAsyncUncaughtExceptionHandler() 메서드 오버라이드
3. SimpleAsyncUncaughtExceptionHandler 또는 AsyncUncaughtExceptionHandler 를 구현한 구현체를 넣어준다.

```java
@Slf4j
public class AsyncEmailExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(final Throwable ex, final Method method, final Object... params) {
        log.error("async email exception message : {}, method : {}, params : {}", ex.getMessage(), method.getName(), params);
    }
}

```

```java
@EnableAsync
@Configuration
public class AsyncThreadPoolConfiguration implements AsyncConfigurer {
    @Bean
    public Executor asyncKakaoEmailTaskExecutor() {
        // ...
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new AsyncEmailExceptionHandler();
    }
}
```

###### 커스텀 구현 로그

AsyncUncaughtExceptionHandler 를 직접 구현했을때의 로그.

![custom-exception-handler](https://raw.githubusercontent.com/choe061/spring-example/master/async-example/src/main/resources/customAsyncUncaughtExceptionHandler-log.png)

## with Transaction

@Async 메소드는 다른 Thread 에서 실행되기 때문에 같은 Transaction 에 묶이지 않는다. 개발시 Thread 분리로 인한 이슈를 고려해야 한다.

