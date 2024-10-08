---
Created: 2024-08-19
---
[Preface](https://docs.spring.io/spring-cloud-stream/docs/current/reference/html/spring-cloud-stream.html#spring-cloud-stream-reference)

[Spring Cloud Stream Kafka Binder Reference Guide](https://docs.spring.io/spring-cloud-stream/docs/current/reference/html/spring-cloud-stream-binder-kafka.html#_reference_guide)

[[kafka/Docker] 도커로 카프카를 띄워보고, 토픽 생성 후 메시지를 보내보자.](https://9hyuk9.tistory.com/92)

[spring cloud stream 정리](https://shuiky.tistory.com/entry/spring-cloud-stream-%EC%A0%95%EB%A6%AC)

# 1. 의존성 추가
    
producer 애플리케이션과 consumer 애플리케이션 모두 추가해준다.
    
``` gradle
implementation 'org.springframework.cloud:spring-cloud-stream'
implementation 'org.springframework.cloud:spring-cloud-starter-stream-kafka'
```

# 2. 메시지 공급 & 소비 함수 작성
    
### 서비스 내에서 만들어지는 데이터를 메시지로 발행
    
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class AuctionSubscriptionServiceImpl implements AuctionSubscriptionService {

	private final AuctionSubscriptionRepository auctionSubscriptionRepository;
	private final StreamBridge streamBridge;

	@Override
	@Transactional
	public void subscribeAuction(AuctionSubscribeRequestDto auctionSubscribeRequestDto) {

		// 서비스 로직
		
		// StreamBridge로 이벤트 발행
		streamBridge.send("auctionSubscription", AuctionSubscriptionMessage.builder()
			.auctionUuid(auctionSubscribeRequestDto.getAuctionUuid())
			.subscribeState(SubscribeState.SUBSCRIBE)
			.eventTime(LocalDateTime.now())
			 .build());
	}
}
``` 

```java
@Getter
@Setter
@Builder
@ToString
public class AuctionSubscriptionMessage {

	private String auctionUuid;
	private SubscribeState subscribeState;
	private LocalDateTime eventTime;
}
```
    
```yaml
spring:
  cloud:
	function:
	  definition: graalSupplier
	stream:
	  bindings:
		graalSupplier-out-0:
		  destination: uppercase-in-0

	  kafka:
		binder:
		  brokers: localhost:9092

  integration:
	poller:
	  fixed-delay: 3000 # 3초 간격으로 polling(공급 함수 실행)
	  max-messages-per-poll: 2 # polling당 2개의 메시지 발행
encrypt:
  key: ${ENCRYPT_KEY}
```
    
- **`spring.cloud.stream.bindings.<bindingName>.destination=<topicName>`**
- spring cloud stream은 기본적으로 1초마다 `Supplier`호출을 트리거하는 폴링을 수행한다. 즉, 매초마다 하나의 메시지를 생성하고 각 메시지는 binder에 의해 노출된 출력 destination으로 전송된다.
	- `spring.integration.poller.fixed-delay`: 기본 poller의 밀리초 단위 고정 지연 시간(default 1000)
	- `spring.integration.poller.max-messages-per-poll`: 기본 poller의 각 polling 이벤트에 대한 최대 메시지 개수(default 1). 2이상이면 polling당 생성된 메시지는 중복되지 않는다.
- `spring.cloud.function.definition = Function;Function;...`: Spring Cloud Function 컨텍스트에서 활성화할 함수들을 묶어 함수형 bean을 지정한다. 함수형 bean은 destination에 binding한다. 하나만 bean으로 등록되있다면 자동 검색되어 설정을 안해도 되지만, 만약 bean이 메시지 처리 이외의 목적일 경우 자동 검색으로 인해 자동 바인딩될 수 있어 명시하는게 좋다.
- 입력과 출력 토픽이 같으면 함수가 처리한 메시지가 다시 자신에게 입력으로 들어올 수 있어 무한 루프에 빠질 위험이 있다.
- **Spring Cloud Stream은 입력과 출력 binding 이름은 기본적으로 `-in-0`, `-out-0`접미사를 붙여 생성한다.**
- `spring.cloud.stream.function.bindings.uppercase-in-0=input` 처럼 descriptive한 바인딩 이름은 암시적이므로 `spring.cloud.stream.function.bindings.uppercase-in-0=destination=my-topic`같은 명시적 바인딩 이름에 매핑하면 잘못된 방향을 만들수 있고, 이후 모든 구성 속성이 이 바인딩이 어떤 함수에 해당하는지 알아야한다.
    
### API 호출로 받아오는 데이터를 메시지로 발행
    
서비스 내에서 만들어지는 데이터가 아닌, API 호출로 클라이언트에서 받아오는 데이터를 메시지로 발행해야 할때는 다음과 같이 작성할 수 있다.
    
`StreamBridge`를 사용한다.
    
```java
@SpringBootApplication
@Controller
public class WebSourceApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebSourceApplication.class, "--spring.cloud.stream.output-bindings=toStream");
	}

	private final StreamBridge streamBridge;

	@RequestMapping
	@ResponseStatus(HttpStatus.ACCEPTED)
	public void delegateToSupplier(@RequestBody String body) {
		System.out.println("Sending " + body);
		streamBridge.send("toStream", body);
	}
}
```
    
- `StreamBridge`에 `Function` bean이 포함되있기 때문에 `Suplier`없이 메시지를 발행할 수 있다.
	
- `StreamBridge`는 `send()`를 처음 호출할때 binding이 존재하지 않으면 binding을 자동으로 생성해 캐싱하고, 존재하면 기존 binding이 사용된다.
	
- 동적 destination들이 많으면 캐싱으로 인해 메모리 누수가 발생할 수 있다.
	
- 기본 캐시 크기인 10을 초과하면 기존 binding이 퇴거되어 다시 만들어져야 하므로 약간의 성능 저하가 발생할 수 있다.
	
- `spring.cloud.stream.dynamic-destination-cache-size` 속성으로 캐시 크기를 조정할 수 있으나, 토픽 관리를 위해 output binding을 명시하는게 좋을 것 같다.
	
- 초기화 시점에 출력 binding을 미리 생성하려면 `spring.cloud.stream.output-bindings` 속성을 지정해야한다.
	
- `StreamBridge.send(String bindingName, @Nullable String binderType, Object data, MimeType outputContentType)`
	- `outputContentType`: data의 타입을 지정하거나 data를 `Message`로 보내면 타입이 자동 적용된다.
	- `binderType`: 여러 종류의 미들웨어(kafka, rabbitmq)를 사용할때 특정 binder를 지정할 수 있다. `spring.cloud.stream.output-bindings`속성이 사용되었거나 binding이 이미 다른 binder에서 생성된 경우 `binderType`은 무시된다.

- `StreamBridge`는 MessageChannel을 사용하여 output binding을 설정하므로 데이터를 보낼때 채널 인터셉터를 활성화할 수 있다.
	
- `@GlobalChannelInterceptor(patterns = “*”)`로 감지된 모든 채널 StreamBridge에 주입할 수 있다.
        
	```java
	@Bean
	@GlobalChannelInterceptor(patterns = "*")
	public ChannelInterceptor customInterceptor() {
		return new ChannelInterceptor() {
			@Override
			public Message<?> preSend(Message<?> message, MessageChannel channel) {
				...
			}
		};
	}
	```
        
    - 특정 채널만 주입할 수도 있다.
        
        2개의 다른 StreamBridge binding이 존재할때
        
        `streamBridge.send("foo-out-0", message);`
        
        `streamBridge.send("bar-out-0", message);`
        
        ```java
        @Bean
        @GlobalChannelInterceptor(patterns = "foo-*")
        public ChannelInterceptor fooInterceptor() {
            return new ChannelInterceptor() {
                @Override
                public Message<?> preSend(Message<?> message, MessageChannel channel) {
                    ...
                }
            };
        }
        
        ```
        
    
### 메시지 소비
    
```java
@Bean
public Function<String, String> uppercase() {
	return String::toUpperCase;
}

@Bean
public Consumer<String> graalLoggingConsumer() {
	return s -> {
		System.out.println("++++++Received 1:" + s);
	};
}

@Bean
public Consumer<String> toStreamConsumer() {
	return s -> {
		System.out.println("++++++Received 2:" + s);
	};
}
```
    
- `Function`을 `Bean`으로 생성하기만 하면 이미 메시지 처리를 할 수 있는 Spring Cloud Stream 애플리케이션을 작성한 것이다.
- 클래스 경로에 spring cloud stream과 binder dependency, auto-configureation 클래스가 존재하기 때문에 spring cloud stream 애플리케이션이 될 수 있다.
- `Supplier`, `Function`, `Consumer`타입의 Bean은 명명 규칙을 따르는 binding을 트리거하는 메시지 핸들러로 취급한다.
- 위 예시에서 `Fuction`과 `Consumer`는 바운드된 destination에 전송된 이벤트를 기반으로 트리거된다. 즉, 전형적인 이벤트 기반 컴포넌트다. WebFlux를 활용한 reactive consumer도 가능하다.
    
```yaml
spring:
  cloud:
	function:
	  definition: uppercase;graalLoggingConsumer;toStreamConsumer;
	stream:
	  bindings:
		graalLoggingConsumer-in-0:
		  destination: uppercase-out-0
		toStreamConsumer-in-0:
		  destination: toStream
	  kafka:
		binder:
		  brokers: localhost:9092
```
    
- binding 명명 규칙
	- input: `<functionName> + -in- + <index>`
	- output: `<functionName> + -out- + <index>`
- API 호출 공급자에서 output binding을 지정안해도 소비자는 input binding을 명시적으로 지정해야한다. 이때 input binding은 소비 함수명-in-0, destination은 공급자의 `streamBridge.send()`에 지정한 bindingName이다.
- 하나의 소비자는 `,` 로 구분된 여러 destination을 가질 수 있다. 이때 **공백이 있으면 안된다**
- `autoRebalanceEnabled=true(default)`면 kafka가 알아서 인스턴스 간 파티션을 분배하기 때문에 따로 `spring.cloud.stream.instanceCount`와 `spring.cloud.stream.instanceIndex`를 설정하지 않아도 된다.
- 특정 파티션에 대한 메시지가 항상 동일한 인스턴스로 전송되도록하려는 경우 `autoRebalanceEnabled=false`로 설정하고 `instanceCount`와 `instanceIndex`를 직접 설정할 수 있다.
- `spring.cloud.stream.defaultBinder`는 여러 종류의 바인더가 있을때 사용한다. 기본값은 비어있다. 우리는 kafka만 사용하므로 설정할 필요없다.
    
### 서비스로직에서 메시지 소비
    
- 메시지에 소비자가 사용할 데이터(i.e. DTO)가 들어가야하는 경우, **메시지 발행자와 소비자 모두 어떤 데이터인지 알고 있어야 한다.** 아래 예시에서는 발행자가 서비스 로직에서 반환한 TestDto를 메시지에 넣어 발행하고 소비자가 이 메시지에서 TestDto를 받아 서비스 레이어에 바로 넘겨준다. 그러면 서비스 로직에서 이 값을 사용할 수 있다.
        
```java
import java.util.function.Consumer;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@RequiredArgsConstructor
public class ConsumerConfig {

	private final TestService testService;

	@Bean
	public Consumer<TestDto> testMessageConsumer() {
		//return this.testService::processTestDto;
		//또는
		return testDto -> {
			this.testService.processTestDto(testDto);
		};
	}
}
```

- OOOService.java
```java
@Service
public class TestService {

	public void processTestDto(TestDto testDto) {
		// testMessageConsumer에서 메시지로 받은 testDto를 처리합니다.
		System.out.println("testDto: " + testDto.toString());
	}
}
```

- application.yml
```java
spring:
  cloud:
	function:
	  definition: testMessageConsumer;toStreamConsumer;
	stream:
	  bindings:
		testMessageConsumer-in-0:
		  destination: testTopic
		toStreamConsumer:
		  destination: toStream
	  kafka:
		binder:
		  brokers: localhost:9092
```

- TestDto
	```java
	@Getter
	@Setter
	@AllArgsConstructor
	@NoArgsConstructor
	@Builder
	@ToString
	public class TestDto {
		private String id = UUID.randomUUID().toString();
		private String content;
		private LocalDateTime createdAt = LocalDateTime.now();
	}
	```
        

# 메시지 발송 후 후처리

- `PostProcessingFunction`인터페이스를 구현해서 `postProcess`와 다른 메서드를 오버라이딩한다.
    ```java
    private static class Uppercase implements PostProcessingFunction<String, String> {
    
    	@Override
    	public String apply(String input) {
    		return input.toUpperCase();
    	}
    
    	@Override
    	public void postProcess(Message<String> result) {
    		System.out.println("Function Uppercase has been successfully invoked and its result successfully sent to target destination");
    	}
    }
    ```
    
- `postProcess`만 구현하고 다른 후처리 함수와 함께 사용할 수 있다.
    ```java
    private static class Logger implements PostProcessingFunction<?, String> {
    
    	@Override
    	public void postProcess(Message<String> result) {
    		System.out.println("Function has been successfully invoked and its result successfully sent to target destination");
    	}
    }
    ```
    

# 에러 핸들링

- 리액티브 함수(WebFlux)는 개별 메시지를 처리하지 않고 사용자가 제공하는 스트림과 자체 스트림(Flux)를 연결하는 방법을 제공해주므로 메시지 핸들러가 아니다.
    
- Retry Template, dropping failed messages, retrying, DLQ, 구성 속성은 메시지 핸들러(명령형 함수)에만 적용된다. 리액티브 함수는 자체적으로 에러 처리를 지원한다.
    
- 첫번째 에러 핸들러는 단순히 에러 메시지를 기록한다.
    
- 두번째 에러 핸들러는 binder별 에러 핸들러로, binder 시스템에 따라 에러 메시지를 처리한다. 두번째 에러 핸들러를 지정하지 않으면 메시지가 삭제된다.
    
- `Consumer`를 커스텀 에러 핸들러로 사용하면 알림을 보내거나 메시지를 DB에 저장할 수 있다. 단, 기본제공되는 에러 핸들러(후처리)와 상호배타적이다.
```java
@Bean
public Consumer<ErrorMessage> myErrorHandler() {
	return v -> {
		// send SMS notification code
	};
}
```
    
- `Consumer`를 에러 핸들러로 식별하기 위해 속성을 설정해야한다.

```
spring.cloud.stream.bindings.<binding-name>.error-handler-definition=커스텀에러핸들러 명
```

암시적 binding명을 사용하는 경우 끝에 `.`을 붙여야 한다.
```
spring.cloud.stream.bindings.upper.error-handler-definition=myErrorHandler.
```
        
- 핸들러를 `Fuction`으로 선언해도 출력에 대해 아무 작업을 하진 않지만 작동은 한다.
    
- 모든 함수 bean에 대해 단일 에러 핸들러를 사용하려면
    
```
spring.cloud.stream.default.error-handler-definition=커스텀에러핸들러 명
```
    

## Dead Letter Queue

- 가장 일반적인 메커니즘으로, 실패한 메시지를 재처리나 auditing 또는 조정을 위해 DLQ로 보낸다.
```java
@Bean
public Function<Person, Person> uppercase() {
	return personIn -> {
	   throw new RuntimeException("intentional");
	  });
	};
}
```
    
- DLQ할 대상의 이름을 올바르게 지정하기 위해 group 속성을 제공하는게 좋지만, group는 destination 속성과 함께 사용되는 경우가 많아 필수는 아니다.
    
- DLQ로 보내는 최대 재시도 횟수를 1로 지정하면 실패시 재시도 없이 바로 보낸다.
    
```
spring.cloud.stream.bindings.uppercase-in-0.consumer.max-attempts=1
```
    
- kafka의 DLQ 속성
    

## RetryTemplate

- `RetryTemplate`는 spring retry 라이브러리의 일부로, application.yml에서 재시도 횟수, 초기 및 최대 지연 시간 등을 설정할 수 있다.
```yaml
spring:
  cloud:
	stream:
	  kafka:
		binder:
		  consumer-properties:
			enable.auto.commit: false
			auto.offset.reset: latest
		  producer-properties:
			retries: 3
			retry.backoff.ms: 1000

```
    
- 커스텀 retry template를 제공하려면 bean으로 만들어야한다. 그리고 binder에서 사용하는 RetryTemplate를 `@StreamRetryTemplate`로 한정해야한다. `@StreamRetryTemplate`를 달면 bean이 된다.
    
- 더 명확히 커스텀 retry template를 지정하려면 소비자의 속성에서 binding당 커스텀 retry template이름을 지정한다.
```
spring.cloud.stream.bindings.<bindingName>.consumer.retry-template-name=<your-retry-template-bean-name>
```