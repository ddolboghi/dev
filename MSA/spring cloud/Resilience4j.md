# 소개
- Resilience4j는 Hystrix 대신 사용할 수 있는 내결함성 라이브러리
- Resilience4j를 사용하면 메서드에 여러 에너테이션을 정의하여 동일한 메서드 호출에 여러 패턴을 적용할 수 있음
	- 회로 차단기: 요청받은 서비스가 실패할때 요청 중단
	- 재시도: 서비스가 일시적으로 실패할때 재시도
	- 벌크헤드: 과부하를 피하고자 동시 호출하는 서비스 요청 수를 제한
	- 속도제한: 서비스가 한번에 수신하는 호출 수 제한
	- 폴백: 실패하는 요청에 대해 대체 경로를 설정

> [!tip] Resilience4j의 재시도 순서
> Retry(CircuitBreaker(RateLimiter(TimeLimiter(Bulkhead([Retry])))))
> - 패턴을 결합하려고 할때 순서대로 해야함 

Resilience4j 사용하는 이유: MSA에서 서비스의 장애가 전체 시스템의 장애로 번지는 것을 막기 위해 클라이언트 회복성을 구현해야한다. 직접 결함 내성 패턴을 구현하려면 스레드와 스레드 관리에 대한 해박한 지식이 필요하고 이 패턴을 높은 품질로 구현하려면 엄청난 양의 작업이 필요하지만, 스프링부트와 Resilience4j를 사용하면 여러 MSA에서 항상 사용되는 검증된 도구를 사용할 수 있다.
# 의존성
- 스프링부트3.x부터는 `2.1.0`버전 이상을 써야함
- 원하는 패턴을 제공하는 라이브러리들만 골라 사용할 수 있음
- `resilience4j-reactor`는 spring webflux를 사용하는 Reactive 환경에서 Resilience4j를 사용 하기 위해 필요함
```gradle
implementation 'org.springframework.boot:spring-boot-starter-webflux'
implementation 'org.springframework.boot:spring-boot-starter-aop'
implementation 'io.github.resilience4j:resilience4j-reactor'
implementation 'io.github.resilience4j:resilience4j-spring-boot3:2.1.0'
implementation 'io.github.resilience4j:resilience4j-circuitbreaker:2.1.0'
implementation 'io.github.resilience4j:resilience4j-timelimiter:2.1.0'
```
# 구현
254~280p