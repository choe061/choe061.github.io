---
title: "스프링 코어 (IoC/DI)"
permalink: /spring/core
categories: IoC DI POJO AOP
---

## spring core (IoC / DI)

#### IoC (Inversion of Control)

스프링 프레임워크에서는 객체의 생성, 의존관계 설정 등 객체 라이프사이클과 관련된 작업을 코드에서 처리하지 않고, 스프링 컨테이너(IoC 컨테이너)에서 처리한다. 개발자가 작성하는 코드가 아닌, Container 가 객체에 대한 제어권을 가지고 있는 모습을 IoC 라고 한다. 개발자는 스프링 컨테이너에서 생성된 객체를 받아서 사용하는 코드만 작성하면 된다.

> `조영호님의 오브젝트`에서도 관련된 내용이 나온다.
>
> 객체의 생성과 사용을 분리하는 것이 유연한 어플리케이션을 만드는데 좋다는 내용이다. 객체를 직접 생성하면 특정 컨텍스트에 대한 결합도가 증가하는 문제점이 있다. 동일한 클래스 내에서 객체 `생성`과 `사용`이라는 반대되는 목적이 공존해서는 안된다.
>
> ###### 생성과 사용을 분리하는 보편적인 방법
>
> 1. 객체 생성의 책임을 클라이언트로 이동한다.
>    - 현재의 컨텍스트에 관한 결정권을 클라이언트로 옮김으로써 객체를 사용하는 쪽은 특정 컨텍스트에 결합되지 않고 독립할 수 있다.
> 2. 더 나아가. 클라이언트 또한 특정 컨텍스트에 얽메이지 않으려면?
>    - <u>객체 생성만 전담하는 객체</u>를 따로 만든다. → ***Factory class***
>
> 스프링 프레임워크의 IoC / DI 가 생성과 사용을 분리하는 대표적인 예인 것 같다. 개발자는 조금만 설정 해주면 객체 생성을 프레임워크에서 알아서 해주는 것이다. 이름도 BeanFactory, 그리고 BeanFactory 를 상속받는 ApplicationContext.

IoC 컨테이너는 Bean(POJO, 단순 자바 객체)의 라이프사이클을 관리한다. spring framework 의 중요한 의의가 Bean 으로 자바 애플리케이션을 개발하는 것으로, 스프링의 주요 기능은 IoC Container 안에서 POJO 를 구성하고 관리하는 일이다.

IoC 컨테이너는 @Component, @Repository, @Service, @Controller 또는 @Configuration + @Bean 가 붙은 코드를 스캐닝하여 Bean 을 구성한다.

###### Bean 생성 예시

```java
@Configuration
public class CompanyConfiguration {
    @Bean
    public Company company() {
        return new Company("google", 1_000_000_000);
    }
}
```

구성 클래스의 메서드에 @Bean 을 붙이면 메서드 이름과 동일한 이름의 bean 이 생성된다. name 속성을 통해 직접 명시하는 방법도 있다.`예) @Bean(name = "googleCorp")` 빈 이름을 지정하면 메서드 명은 무시된다. 같은 객체를 여러 bean 으로 등록해야하는 경우가 아니면 보통 클래스 이름과 동일하게 빈 이름이 생성되도록 한다.

#### IoC 컨테이너와 Bean 스캐닝

Bean 을 스캐닝하기 위해서는 먼저 IoC 컨테이너를 생성(인스턴스화)하고, Bean 을 등록해야 한다.

###### Spring IoC Container 의 기반

두 package

* org.springframework.beans
* org.springframework.context

 BeanFactory interface

* 어떤 타입의 객체도 다룰 수 있는 향상된 설정 메커니즘 제공
* ApplicationContext 가 BeanFactory 의 sub-interface

###### bean factory / application context

bean factory 는 스프링에서 제공하는 기본 구현체이고, 빈 팩토리와 호환되는 고급 구현체인 application context 가 있다. bean factory 와 application context 가 IoC 컨테이너이다. ApplicationContext 는 BeanFactory 보다 발전된 기능의 하위 인터페이스이므로 호환성도 보장되기 때문에 ApplicationContext 를 사용하는 것이 좋다.

| 상속 구조                                           |
| --------------------------------------------------- |
| BeanFactory<br />         ↑<br />ApplicationContext |

###### Bean 등록 과정

* Spring Container

  * 개발자가 작성한 POJOs 와 설정 Metadata 가 결합되어 컨테이너에 등록된다.
  * 예시) ClassPath 하위에 두 xml 파일을 metadata 로 load 하는것. 두 xml 을 기반으로 bean 을 로딩시키고 bean 들이 서로 의존 관계를 맺는다.

  ```java
  // xml config
  ApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"application.xml", "application-test.xml"});
  // java config
  ApplicationContext context = 
      new AnnotationConfigApplicationContext(SequenceGeneratorConfiguration.class);
  // bean 가져오기
  SequenceGenerator generator = context.getBean(SequenceGenerator.class);
  ```

* ApplicationContext

  * 애플리케이션 컨텍스트가 생성되고 인스턴스화되면 완전히 설정이 완료된 것이고, 실행 가능한 애플리케이션이 된다.

#### Component, Repository, Service, Controller

@Component 를 사용해 bean 을 정의할 수 있다. 하지만 @Component 를 사용하여 DAO 클래스, Service 클래스, Controller 클래스를 작성하지 않는 이유는 ?

우선, **클래스의 용도를 구체적으로 명시**하기 위한 것도 있다. **그런데, Repository 의 경우 `@Repository` 를 붙인 클래스는 발생한 예외를 DataAccessException 으로 감싸 던지므로?! 디버깅에 좋다.**

#### DI(Dependency Injection)

Bean 을 주입받는 방법은 3가지 정도로 정리할 수 있다.

1. setter based injection
2. field injection
3. constructor based injection

###### @Autowired

annotation 을 활용한 방법으로 많이 사용되지만, 이 방법보다 생성자 주입을 많이 사용한다.

참고로 IntelliJ 에서 Autowired 를 사용하며 경고 표시를 해주면서 constructor based injection 을 사용하라고 한다.

###### setter based injection

* 장점이 없다.
* 단점
  1. 불변 객체가 아니다. 중간에 객체를 변경할 수 있다.
  2. setter injection 없이도 해당 객체가 생성될 수 있다. 주입 받는 객체의 메서드를 call 하면 NullPointerException 이 발생한다.
     - 그래서 주입받는 의존성에 대한 not null 체크가 꼭 필요하다.

###### field injection

setter injection 의 단점과 동일하다. 더하여 setter 에서는 외부에서 객체를 주입할 수 있는 방법이 있지만, field injection 은 외부에서 객체를 주입할 수 있는 방법이 스프링 컨테이너를 통한 방법 밖에 없다. 그래서 test code 를 작성할 때도 DI Container 에 의존할 수 밖에 없다.

###### constructor based injection

[Spring core 문서](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-constructor-injection)를 보면 constructor 기반 주입과 setter 기반 주입을 비교하는 부분이 있는데, 결론은 생성자 기반 주입을 추천한다. 왜 @Autowired, setter-based injection 을 비추천하고, 생성자 주입 방법을 추천할까.

#### Constructor-based vs setter-based DI

> 혼합하여 사용할 수 있지만, Spring 팀은 생성자 주입을 선호한다. 생성자 주입을 사용하면, 컴포넌트를 불변 객체로 구현하고 종속성이 not null 을 보장할 수 있다.
>
> 부수적으로는 생성자 인수가 많으면 악취가 나는 코드이므로 해당 클래스가 너무 많은 책임을 가질 가능성이 있으며 적절한 분리를 하여 리펙토링 해야함을 드러낼 수도 있다.
>
> Setter Injection 을 사용한다면, 주로 클래스 내에서 기본 값을 할당할 수 있는 옵셔널한 종속성에만 사용해야 한다. 그렇지 않다면, 해당 종속성을 사용하는 모든 부분에서 null 체크를 해야한다. setter injection 의 한가지 장점은 객체를 나중에 재구성하거나 재주입할 수 있다는 것이다.

하지만, setter injection 의 장점인 객체를 나중에 재구성할 수 있다는 것은 오히려 그 객체가 여러 가지 일을 하여 코드를 파악하는데 어려울 것 같다. 

차라리 의존성을 재구성하지 않고 주입받는 객체를 분리하는 것이 좋을 것 같다. 필드는 final 로 선언하고 생성자 주입을 받으면 생성 이후 객체를 변경할 수 없어 불변 객체가 된다는 장점이 있다. (불변 객체의 장점 : side-effect ↓, code quality ↑)

###### Test Code

Test Code 를 작성할때 field injection 은 DI Container 에 의존하여 test code 에서도 DI Container 에 의존해야한다. 즉, test code 를 수행하기 위해 스프링 컨테이너를 띄워야 한다는 것이다. 스프링 컨테이너를 띄우면 시간이 오래걸리고, 전체 테스트 코드를 돌리는데 시간이 늘어나는 문제점이 생긴다.

반면, constructor injection 은 DI Container 에 의존하지 않고도 쉽게 단위 테스트를 작성할 수 있다. 필요한 의존성들을 mock 객체로든 어떻게든 생성하여 생성자의 인자로 넣어주면 스프링 컨테이너를 띄우지 않고 test code 를 수행할 수 있다.

###### 런타임 문제를 구동 시점에 알려준다.

추가적으로 생성자 주입 방식은 객체의 순환 참조도 스프링 구동 시점에 BeanCurrentlyCreationExeption 를 발생시켜 잡아준다. setter 나 field injection 방식은 런타임 방식에 에러가 발생하여 StackOverflowError 가 발생한다. 원래는 순환 참조의 설계가 잘못되었다고 할 수 있지만, 그 잘못을 스프링 구동 시점에 알려준다는 것이 더 좋은 것 같다.

스프링 4.3 부터는 생성자에 @Autowired 를 붙이지 않아도 주입이 가능하다.

#### Bean Scope

###### 의존성 주입이 어느 시점에 되는가?

Bean 의 scope 에 따라 다르다. spring framework 에서 Bean 의 default scope 는 `singleton` 이다. 싱글톤 스코프에서는 애플리케이션이 처음 구동될때, Bean 들이 IoC 컨테이너에 등록된다. 그때 의존성이 주입되고 의존 관계가 형성된다.
