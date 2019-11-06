---
title: "Spring Boot AutoConfiguration"
permalink: /spring-boot/autoconfiguration
categories: spring-boot AutoConfiguration
---

## spring 의 @Conditional
spring boot 의 AutoConfiguration 은 `@Conditional` 에 기반한다. 스프링 기반의 애플리케이션을 개발할때 조건부적으로 beans 을 등록할 필요가 생긴다. 예를들어 로컬 환경에서는 dev db, 프로덕션 환경에서는 production db 에 붙어야 하는 경우가 있다. 그때 properties files 에 database connection 정보를 넣고, 환경에 따라 적절한 파일을 사용할 수 있다. 그런데 각각의 환경에 따라 변경점이 필요하다면 configuration 을 변경할 수 있어야 한다.

Spring 3.1 에서 `Profiles` 개념이 도입되어 동일한 type 의 beans 을 등록할 수 있고 하나 이상의 profiles 에 연관시킬 수 있다. 그리고 application 을 실행시킬때 원하는 profiles 만 활성화시킬 수 있고, 활성화된 profiles 에 연관된 beans 만 등록된다.

Spring 4 에서 `@Conditional` 이 도입되어 Spring beans 를 조건에 따라 등록할 수 있게 되었다.

## spring boot AutoConfiguration
#### @EnableAutoConfiguration
이 어노테이션을 통해 스프링 부트는 AutoConfiguration 설정이 되고, `@SpringBootApplication` 의 내부를 보면 이 주석이 포함되어 있다. 또 @SpringBootApplication 에는 `@ComponentScan` 이 있어 classpath 에서 components 들을 스캐닝하고 Condition 에 매칭되는 beans 을 등록함으로써 ApplicationContext 의 자동 구성이 가능한 것이다.

#### AutoConfiguration 구현 방법
AutoConfiguration 클래스는 @Configuration 으로 설정 클래스로 만들고, @EnableConfigurationProperties 로 사용자 정의 특성을 받아 하나 이상의 Conditional bean 등록 메소드를 바인딩한다.

#### 1. 여러 속성을 Properties 클래스로 받는 방법

```yaml
company:
  name: 'bk'
  value: 1_000_000
```

```java
@Getter
@RequiredArgsConstructor
@ConstructorBinding
@ConfigurationProperties(prefix = "company")
public class CompanyProperties {
    private final String name;
    private final long value;
}
```

```java
//@EnableConfigurationProperties(CompanyProperties.class)
// boot 2.2 부터 생략해도 classpath 기반으로 스캐닝함, 2.1 이하에서는 추가해야함
@Configuration
@RequiredArgsConstructor
public class StudyAutoConfiguration {
    private final CompanyProperties properties;
    @Bean
    public Company company() {
        return new Company(properties.getName(), properties.getValue());
    }
}
```

#### 2. 조건(Condition)에 따라 properties 를 다르게 받는 방법

`@Conditional` 을 사용하여 조건에 따라 Bean 을 등록할 수 있다. @Conditional 의 속성 값으로 `Condition` interface 를 구현한 클래스를 등록해야 한다.

```java
public class CompanyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context,
                           AnnotatedTypeMetadata metadata) {
        // return 값이 true 이면 bean 이 등록된다.
        return context.getEnvironment()
            		  .containsProperty("company");
    }
}
```

```java
@Configuration
@RequiredArgsConstructor
public class StudyAutoConfiguration {
    private final CompanyProperties properties;
    @Bean
    @Conditional(CompanyCondition.class)
    public Company company() {
        return new Company(properties.getName(), properties.getValue());
    }
}
```

#### 3. 환경(profile)에 따라 properties 를 다르게 받는 방법

profile 에 따라 configuration 정보를 다르게 받는 방법도 결국 내부적으로 2번의 방법(Conditional)을 사용하여 구현하고 있다.

```java
// ...
@Conditional(ProfileCondition.class)
public @interface Profile {
    String[] value();
}
```

아래 처럼 `@Profile` 을 이용하여 dev, local 각각 다른 설정이 가능하다.

```java
@Profile("dev")
@Configuration
public class StudyAutoConfiguration {
    //
}

@Profile("local")
@Configuration
public class StudyAutoConfiguration {
    //
}
```

#### 4. Spring boot 가 자동으로 설정 정보를 가져오는 방법

spring boot 시작 시 application.yml 같은 설정 파일을 읽어서 설정 정보에 대한 값이 없으면 디폴트 값으로, 있으면 yaml 파일의 값을 기준으로 spring bean 을 생성한다.

###### EnableAutoConfiguration

우선 @SpringBootApplication 내에 @EnableAutoConfiguration 이 포함되어 있다. 이 어노테이션으로 스프링 부트가 자동으로 설정 정보를 가져올 수 있게 된다. 내부에 PROPERTY 속성이 있는데, `spring.boot.enableautoconfiguration` 이라는 일종의 스프링 부트의 자동 설정 정보를 담는 key ? 가 있고, `EnableAutoConfigurationImportSelector.class` 에 스프링 부트 자동 설정 기능을 담당하는 클래스가 지정되어 있다.

```java
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    //...
}
```

###### EnableAutoConfigurationImportSelector

이 클래스는 `DeferredImportSelector` 인터페이스를 상속받는데, `@Configuration` 을 받아 활성화하는 기능의 인터페이스이다. 첫 번째 selectImports 메소드가 구현한 메소드이다.

두 번째 메소드는 Classpath 의 모든 라이브러리의 `META-INF/spring.factories` 위치의 파일에서 설정 파일 리스트를 읽어온다. 스프링 부트 라이브러리를 확인해보면 META-INF 패키지 아래 spring.factories 파일이 있다. 그 파일에는 설정 클래스 리스트가 들어가 있다.

```java
public class EnableAutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
	@Override
    public String[] selectImports(AnnotationMetadata metadata) {
        //... return 되는 배열은 설정 클래스의 패키지와 클래스 리스트
    }
    //...
    protected List<String> getCandidateConfigurations(...) {
        return SpringFactoriesLoader.loadFactoryNames(
            getSpringFactoriesLoaderFactoryClass(),
            getBeanClassLoader());
    }
}
```

 예를 들어 spring-boot-actuator 라이브러리를 디펜던시에 추가하면 `spring-boot-actuator` 와 `spring-boot-actuator-autoconfigure` 두개의 디펜던시가 추가된다. META-INF 폴더 아래 `spring.factories` 와 `spring-autoconfigure-metadata.properties` 가 중요하다.

![image](https://raw.githubusercontent.com/choe061/choe061.github.io/master/assets/images/study/autoconfiguration.png)

* spring.factories

  * @Configuration 클래스 파일들이 나열되어 있다.
  * AutoConfiguration 도 있고, 그냥 Configuration 도 있는듯

* spring-autoconfigure-metadata.properties

  * 이 파일에 자동 설정을 해주기 위한 디폴트 설정 정보가 들어가 있다.

  * 설정 Key=Value 형식, 아래 예시 처럼 디폴트 Value 가 있는 경우도 있고 없는 경우도 있다.

    ```properties
    org.springframework.boot.actuate.autoconfigure.redis.RedisReactiveHealthIndicatorAutoConfiguration=
    org.springframework.boot.actuate.autoconfigure.metrics.web.servlet.WebMvcMetricsAutoConfiguration.AutoConfigureAfter=org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration,org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleMetricsExportAutoConfiguration
    ```

###### Conditional 적용

몇 가지만 골라서 봤지만, spring-boot-autoconfigure 디펜던시의 spring.factories 파일에 나열된 설정 클래스에 Conditional 이 굉장히 많이 사용되어 있었다. 아래는 Actuator 의 설정 클래스 중 하나인데, 

* @ConditionalOnWebApplication : 현재 웹 애플리케이션인 경우만 bean 을 생성
* @AutoConfigureAfter(EndpointAutoConfiguration.class) : 
* @EnableConfigurationProperties(WebEndpointProperties.class) : WebEndpointProperties 클래스를 통해 설정 파일의 management.endpoints.web 을 key 로 하는 설정 값들을 가져온다.

```java
@Configuration
@ConditionalOnWebApplication
@AutoConfigureAfter(EndpointAutoConfiguration.class)
@EnableConfigurationProperties(WebEndpointProperties.class)
public class WebEndpointAutoConfiguration {
}
/*------------------------------------------------------------------------------*/
@ConfigurationProperties(prefix = "management.endpoints.web")
public class WebEndpointProperties {
}
```

####   DataSource 를 설정하는 예시

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
public class DataSourceAutoConfiguration {
  // ...
  @Configuration
  @ConditionalOnClass(DataSourceAutoConfiguration.EmbeddedDataSourceCondition.class)
  @Import(EmbeddedDataSourceConfiguration.class)
  protected static class EmbeddedConfiguration {
  }
  
  
}
```

#### 그 외 @ConditionalOnClass, @ConditionalOnProperty, @ConditionalOnMissingBean 등...

위 어노테이션들도 @Conditional 을 포함하여 각각의 해당 조건이 일치하는 경우에만 ApplicationContext 에 Bean 으로 등록된다.