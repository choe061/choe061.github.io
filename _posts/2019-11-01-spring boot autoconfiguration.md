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

#### 2. 환경(or 조건)에 따라 properties 를 다르게 받는 방법

```java

```



#### 다른 예시

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

#### @ConditionalOnClass, @ConditionalOnProperty, @ConditionalOnMissingBean 등...

위 어노테이션은 @Conditional 을 포함하여 각각의 해당 조건이 일치하는 경우에만 ApplicationContext 에 Bean 으로 등록된다.