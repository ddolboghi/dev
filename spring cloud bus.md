---
Created: 2024-08-19
sticker: emoji//1f68c
---
bus는 동일한 애플리케이션이 여러 인스턴스를 가질때 모든 인스턴스에 변경사항을 kafka로 전파해주는 역할을 한다.

그리고 변경사항을 반영하려면 `POST /actuator/busrefresh`를 호출해야한다.

하지만 actuator 대신 spring cloud config monitor를 사용하면 별도의 webhook 애플리케이션 개발 없이 config 서버에 바로 요청할 수 있다.

깃헙 웹훅 설정으로 config 설정을 변경할때마다 자동으로 실행되게하자

# 1. kafka 실행
    
# 2. config 서버에 의존성 추가
    
```
implementation 'org.springframework.cloud:spring-cloud-config-monitor'
implementation 'org.springframework.cloud:spring-cloud-starter-stream-kafka'
```
    
# 3. config 서버 application.yml에 kafka, bus 설정 추가
    
```yaml
spring:
  #kafka:
   # bootstrap-servers:
	#  - <kafka 서버 host:port> # kafka 주소

  cloud:
	  stream:
		  binder:
			brokers: <kafka 서버 host:port>
	bus:
	  enabled: true
```
만약 spring-cloud-stream-kafka 대신 spring-kafka를 사용한다면 주석친 부분만 작성하면 된다.
    
# 4. 서비스 애플리케이션에 의존성 추가
    
```
implementation 'org.springframework.cloud:spring-cloud-starter-bus-kafka'
```
    
# 5. 서비스 application.yml에 kafka 설정 추가
    
```yaml
spring:
	kafka:
		bootstrap-servers: <kafka 서버 host:port>
	
```
    
# 6. 깃헙 레포지토리에 웹훅으로 `config서버 배포 주소/monitor` 추가
    ![[Pasted image 20240819164921.png]]

    
# 7. 서비스Application.java에 `@RefreshScope` 달기
    
# 8. 서비스 application.yml 수정
    
```yaml
spring:
  application:
	name: member-service
  profiles:
	active: none
  config:
	import: optional:configserver:<config 서버 주소>
  cloud:
	config:
	  profile: ${PROFILE}
	stream:
	  kafka:
		binder:
		  brokers: <KAFKA 배포 주소> # 배포 시 INTERNAL 주소로 변경
```
    
- KAFKA INTERNAL 주소는 같은 인스턴스 내에 있어야 사용가능하다.
- spring-kafka를 사용한다면 `spring.cloud.stream.kafka.binder.brokers` 대신 `spring.kafka.bootstrap-servers`를 작성해주자.

이제 config 설정 수정후 github에 push만 하면 애플리케이션 재시작 없이 자동으로 반영된다.

참고
[Spring Cloud Config, Spring Cloud Bus & Kafka | How to set up automatic updates of your configuration](https://dev.to/vrnsky/spring-cloud-config-spring-cloud-bus-kafka-how-to-set-up-automatic-updates-of-your-configuration-5d09)

[Spring Cloud Config & Cloud Bus 정리 - Yun Blog | 기술 블로그](https://cheese10yun.github.io/spring-config-client/#null)