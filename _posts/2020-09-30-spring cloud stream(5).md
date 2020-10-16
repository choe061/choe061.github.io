---
title: "Spring Cloud Stream Test"
permalink: /spring/spring-cloud-stream/5
categories: spring spring-cloud-stream
---

# spring-cloud-stream(5) How to test

## Spring Integration Test Binder dependency

spring-cloud-stream-test-support 는 unit testing 처리는 가능하지만 가볍게 설계되어 binder(rabbit, kafka...) 와 함께 동작하는 추가적인 integration testing 이 필요하다. 현재 spring cloud stream 에서는 deprecating 하려고 한다.

* 추가
  * `testImplementation 'org.springframework.cloud:spring-cloud-stream:3.0.8.RELEASE:test-binder'`
* 삭제
  * spring-cloud-stream-test-support
  * 충돌 가능성이 있기 때문에 두 의존성을 함께 사용하는 것은 지양

---

#### Example code

```java
public Function<String, String> uppercase() {
    return input -> input.toUpperCase();
}
```

---

#### Test 1

```java
@Test
public void testUpperCase() {
    assertThat(new ToUpperCaseProcessor().transform("foo")).isEqualTo("FOO");
}
```

* Only Unit Test
* 추가적인 Integration Test 작성이 필요

---

#### Test 2

```java
@SpringBootTest(classes = SampleStreamTests.SampleConfiguration.class)
@ExtendWith(SpringExtension.class)
public class SampleStreamTests {
	@Autowired
	private InputDestination input;
	@Autowired
	private OutputDestination output;

	@Test
	public void testEmptyConfiguration() {
		this.input.send(new GenericMessage<byte[]>("hello".getBytes()));
		assertThat(output.receive().getPayload()).isEqualTo("HELLO".getBytes());
	}

	@SpringBootApplication
	@Import(TestChannelBinderConfiguration.class)
	public static class SampleConfiguration {
		@Bean
		public Function<String, String> uppercase() {
			return v -> v.toUpperCase();
		}
	}
}
```

---

#### Test 3

* Test 2 보다 세밀한 테스트를 원하는 경우
  1. multiple bindings and/or multiple inputs/outputs
  2. 심플한 경우지만 destination 을 지정하고 싶을때

```java
@EnableAutoConfiguration
public static class MyTestConfiguration {
	@Bean
	public Function<String, String> uppercase() {
			return v -> v.toUpperCase();
	}
}

@Test
public void sampleTest() {
	try (ConfigurableApplicationContext context = new SpringApplicationBuilder(
				TestChannelBinderConfiguration.getCompleteConfiguration(
						MyTestConfiguration.class))
				.run("--spring.cloud.function.definition=uppercase")) {
		InputDestination source = context.getBean(InputDestination.class);
		OutputDestination target = context.getBean(OutputDestination.class);
		source.send(new GenericMessage<byte[]>("hello".getBytes()));
		assertThat(target.receive().getPayload()).isEqualTo("HELLO".getBytes());
	}
}
```

