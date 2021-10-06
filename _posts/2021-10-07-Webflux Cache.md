---
title: "Spring Webflux - Cache"
permalink: /spring/webflux/cache
categories: spring webflux cache async-cache
---

# Webflux & Spring Cache Abstraction

* 기존(서블릿 스택) 스프링 환경에서는 Spring Cache Abstraction 이 잘되어 있어서 정말 쉽게 적용할 수 있다. 원하는 캐시 구현체를 설정해주고, Spring 에서 지원하는 @Cacheable annotation 을 사용하면 된다.
* 그런데 spring webflux stack 에서는 기본적으로 aop 가 디펜던시에 없고, spring cache abstraction 을 활용할 수 없다.

## Spring Webflux 에서 로컬 캐시가 어려운 점

#### AOP

* 일단 spring-boot-starter-web 종속성을 추가했을땐 aop dependency 가 포함되어 있지만, spring-boot-starter-webflux 종속성에는 aop 가 없다. 그래서 기존에 spring 환경에서 aop 기반으로 동작하는 @Cacheable 이 동작하지 않는다. (@Transactional, @Async, @Retryable 모두 동작하지 않는다.)
* 만약 annotation 방식을 webflux 에서도 사용하려면 기존의 Spring Cache Abstraction 은 이용할 수 없고, custom Cacheable annotation 과 Aspect 를 직접 구현해야 한다.

#### Cache API

* 기존의 Cache API 는 reactive 에 어울리지 않는 blocking 모델들이다. 그대로 webflux 환경에서 사용한다면, blocking 코드를 넣는 것이다.
* Reactive 환경에서 사용하려면 reactive type 을 지원하는 Cache 구현체가 필요하다. 뒤에서 설명하겠지만, Caffeine 에서 reactive 환경에서 사용할 수 있는 AsyncCache 를 지원한다.

#### Mono, Flux

* Mono, Flux 에도 cache 기능을 지원하긴 한다.

  ```kotlin
  Mono.just("data")
      .cache(Duration.ofSeconds(10))
  ```

* 이런식으로 TTL 까지 파라미터로 넣어줄 수도 있다. 하지만 일반적으로 우리가 생각하는 캐시로써 사용할 수 없다. 위에 작성한 예제에서 Mono 객체를 Spring Cache 와 조합하여 사용하면 "data" 가 저장되는 것이 아니라 Mono 객체의 레퍼런스 주소가 저장된다. 그런데 Mono, Flux 는 구독(subscribe)하기 전까지는 데이터를 받아볼 수 없다. 이 간극으로 인해 Spring Cache Abstraction 과 Mono, Flux 는 기존 처럼(web servlet stack) 동작하지 않는다. 그리고 Mono 객체에 TTL 을 위 처럼 설정하고, Spring CacheManager 에 TTL 를 설정한다고 했을때 그 두 TTL 이 정확히 일치하지 않을 수도 있다.

## Spring, Caffeine Github 에 올라온 Q&A

* https://github.com/spring-projects/spring-framework/issues/26713
  * Q : Spring cache 환경에서 Async caffeine cache 를 적용할 수 있나요?
  * A-1 : Spring 개발자의 답변
    * Cache API 를 직접 사용한다면, blocking 되기 때문에 reative type 과 맞지 않다.
  * **A-2 : Caffeine cache 개발자의 답변**
    * reactive application 을 위한 `AsyncCache` interface 가 있고, Caffeine cache 는 구현체를 지원한다. Caffeine 에서는 CompletableFuture 를 사용하기 때문에 reator 에서 쉽게 적용할 수 있다.
    * 하지만 Spring Cache Abstraction 에 아직 연동하지 못하기 때문에 native api 를 직접 사용해야 한다.
  * 즉, Async 를 지원하는 캐시 구현체를 사용해야 진짜 reactive application 에서 사용할 수 있는 캐싱 방법이다.
* https://github.com/ben-manes/caffeine/discussions/518
  * 위 spring-framework 에 등록된 이슈에서 나온 동일한 이슈인듯
  * Q : AsyncCache.. is thread-safe?
  * A : Yep, `AsyncCache` is thread-safe.

## 구현 방법

#### CacheMono, CacheFlux & AsyncCache

* [Reactor-extra](https://projectreactor.io/docs/extra/snapshot/api/overview-summary.html#overview.description) 의 CacheMono(CacheFlux) 와 [Caffeine](https://github.com/ben-manes/caffeine) 의 AsyncCache 를 조합해서 캐시를 적용해볼 수 있다.
* webflux 환경에서 kotlin, coroutine 을 사용한다고 하더라도 Filter 등 Spring interface 를 구현해야 할때 suspend fun 을 사용할 수 없기 때문에 그때 CacheMono, CacheFlux 를 사용해야한다.

#### dependency

```kotlin
implementation("io.projectreactor.addons:reactor-extra")
implementation("com.github.ben-manes.caffeine:caffeine")
```

* reactor-cache 를 위해 `reactor-extra` 을 추가한다.
* AsyncCache 를 위해 `caffeine` 을 추가한다.
* Caffeine 의 AsyncCache 를 직접 native api 호출 방식으로 사용할 것이기 때문에 spring-boot-starter-cache 는 필요없다.

#### CacheMono, CacheFlux 에 대해 알아보기

* CacheMono 의 lookup, onCacheMissResume, andWriteWith

```kotlin
fun getUser(key: Long): Mono<User> {
  // 아래 코드에서 cacheReader, cacheWriter 구현
  return CacheMono.lookup(/* cacheReader() */, key)
  .onCacheMissResume { userService.getUser(key) }
  .andWriteWith(/* cacheWriter() */)
}
```

* lookup
  * key 에 해당하는 cache data 를 찾는 작업
* onCacheMissResume
  * lookup 에서 발견된 cache 가 없을 경우 origin data 를 구하기 위한 작업 호출
* andWriteWith
  * onCacheMissResume 다음에 호출되어 origin data 를 cache 에 put

#### Caffeine 의 AsyncCache 설정하기

* AsyncCache Bean 등록

  ```kotlin
  @Configuration
  class CaffeineAsyncCacheConfig {
    // String type 의 key
    // User 를 Value 로 캐싱하고 싶을때 아래와 같이 설정
    @Bean
    fun asyncUserCache(): AsyncCache<String, User> {
      return Caffeine.newBuilder()
        .expireAfterWrite(30, SECONDS)
        .maximumSize(10_000)
      	.recordStats()	// 아쉽지만 actuator 와 연동은 안되는 것 같다.
        .buildAsync()
    }
  }
  ```
  * 기존 Caffeine 객체를 Bean 으로 등록하는 과정과 설정 코드는 동일한데, 마지막에 `buildAsync()` 를 추가하면 Caffeine 의 AsyncCache 로 생성된다.

  * 그리고 기존에는 CacheManager 에 구현체(Caffeine, EhCache 등등..)만 설정해서  Spring Cache 를 이용할 수 있었는데, Reative 방법은 아직 지원하지 않아서 Caffeine 의 AsyncCache 는 직접 native API 를 호출해야한다.

  * 한 가지 아쉬운 점은 actuator 를 통해 cache hit rate 보는 것이 캐시 설계, 튜닝을 할때 좋은 참고 자료가 될 것 같았는데, CacheManager 대신 native api 를 직접 사용해서인지 actuator 에서 보여지지 않는다. 코드에서 직접 호출하여 확인할 수 있는 방법은 있긴하다. 그런데 synchronous() 로 view 를 가져올 수 있는데 프로덕션 환경에서 사용해도 될지는 모르겠다. (일단 쓰지 말란 말은 없는 것 같다.)

    * [Caffeine 개발자의 stackoverflow 답변 참고](https://stackoverflow.com/a/56828561/10958783)

    ```kotlin
    fun getCacheStats() {
    	val stats = asyncUserCache.synchronous().stats()
      println(stats)
    }
    
    // CacheStats{hitCount=5, missCount=1, loadSuccessCount=1, loadFailureCount=0, totalLoadTime=2113585, evictionCount=1, evictionWeight=1}
    ```

* AsyncCache 라이브러리 코드를 보면, Parameter 와 Return Type 을 보면 Java8 에서 나온 CompletableFuture 를 사용하기 때문에 Async 로직을 작성할 수 있다.

  ```java
  public interface AsyncCache<K, V> {
  	CompletableFuture<V> getIfPresent(@NonNull @CompatibleWith("K") Object key);
    
    void put(@NonNull K key, @NonNull CompletableFuture<V> valueFuture);
  
    //...
  }
  ```

#### Application 기능 구현

* Application code 에 캐시 기능 구현하기 (UserCacheService class)

  * AOP, Annotation 방식으로 구현하지 않고, `Decorator pattern` 으로 UserService 를 wrapping 한 UserCacheService 를 만드는 방식으로 구현했다. 그 이유는... aop dependency 를 추가하고 직접 custom annotation 을 추가하면 어렵지 않게 구현할 수도 있을 것 같고 구글링해보면 이런 방법의 예제도 나온다. 그런데 spring-framework 개발팀 하지 않은 이유가 있지 않을까?라는 생각으로 직접 caffeine native api 를 사용하는 방식으로 구현했다.

  ```kotlin
  import com.github.benmanes.caffeine.cache.AsyncCache
  import org.springframework.stereotype.Service
  import reactor.cache.CacheMono
  import reactor.core.publisher.Mono
  import reactor.core.publisher.Signal
  import java.util.concurrent.CompletableFuture
  
  @Service
  class UserCacheService(
    private val asyncUserCache: AsyncCache<String, User>,
    private val userService: UserService,
  ) {
  
    fun getUser(userId: Long): Mono<User> {
      return CacheMono.lookup(cacheReader(), userId.toString())
        .onCacheMissResume { userService.getUser(userId) }
        .andWriteWith(cacheWriter())
    }
  
    private fun cacheReader(): (key: String) -> Mono<Signal<out User>> {
      return { key ->
        val value = asyncUserCache.getIfPresent(key)
        if (value == null) {
          Mono.empty()
        } else {
          Mono.fromFuture { value }
            .map { Signal.next(it) }
        }
      }
    }
  
    private fun cacheWriter(): (key: String, value: Signal<out User>) -> Mono<Void> {
      return { key, value ->
        Mono.fromRunnable {
          asyncUserCache.put(key, CompletableFuture.completedFuture(value.get()))
        }
      }
    }
  }
  ```

  ```kotlin
  @Service
  class UserService {
    private val log = LoggerFactory.getLogger(javaClass)
  
    fun getUser(userId: Long): Mono<User> {
      val user = User(userId, "bk")
      log.info("Call getUser method. $user")
      return Mono.just(user)
    }
  }
  ```


