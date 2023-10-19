# gradle 프로젝트를 springboot 프로젝트로 만들기
1. build.gradle 파일 수정하기
```gradle
//1
plugins {  
    id 'java'  
    id 'org.springframework.boot' version '3.0.2'  
    id 'io.spring.dependency-management' version '1.1.0'  
}  

//2
group = 'me.siwoli'  
version = '1.0-SNAPSHOT'  
sourceCompatibility = '17'  

//3
repositories {  
    mavenCentral()  
}  

//4
dependencies {  
    testImplementation 'org.springframework.boot:spring-boot-starter-test'  
    implementation 'org.springframework.boot:spring-boot-starter-web'  
}  
  
test {  
    useJUnitPlatform()  
}
```
	1. plugins : springboot 3.0.2 버전 플러그인과 스프링의 의존성을 자동으로 관리하는 spring.dependency-management 추가
	2. group : 프로젝트 그룹 이름, 프로젝트 버전, 자바 버전
	3. repositories : 의존성을 받을 저장소 지정, 기본 값 mavenCentral()
	4. dependencies : 프로젝트에 필요한 기능의 의존성 추가 

2. src/main/java/그룹이름패키지 하위에 "그룹_이름.프로젝트_이름" 형식으로 패키지 생성
3. 스프링 부트 실행할 용도의 "프로젝트_이름Application"클래스 생성

# spring boot vs spring
| |spring|spring boot|
|---|---|---|
|목적|쉬운 엔터프라이즈 애플리케이션 개발|더 쉽고 빠른 스프링 개발|
|설정 파일|수동 구성|자동 구성|
|XML|일부 파일은 XML로 직접 생성하고 관리|사용 안함|
|인메모리 데이터베이스 지원|X|O|
|WAS|수동 설정|내장 WAS 제공하므로 별도 설정 필요 없음|

* 빌드 구성을 단순화하는 springboot-starter제공
* jar을 이용해서 자바 옵션만으로도 배포 가능
* spring actuator로 애플리케이션 모니터링 및 관리

# spring-boot-starter-{작업유형}
build.gradle에 필요한 기능을 간편하게 추가할 수 있음

|스타터|설명|
|:---|:---|
|spring-boot-starter-web|springMVC를 사용해 RESTful웹 서비스를 개발할때 필요한 의존성 모음|
|spring-boot-starter-test|스프링 애플리케이션을 테스트하기 위해 필요한 의존성 모음|
|spring-boot-starter-validation|유효성 검사를 위해 필요한 의존성 모음|
|spring-boot-starter-actuator|모니터링을 위해 애플리케이션에서 제공하는 다양한 정보를 제공하기 쉽게 하는 의존성 모음|
|spring-boot-starter-data-jpa|ORM을 사용하기 위한 인터페이스의 모음인 jpa를 더 쉽게 사용하기 위한 의존성 모음|

# 자동 구성
* 스프링 부트는 내가 구성하지 않은 부분을 자동으로 구성해주기 때문에 추후 이 부분을 확인할 상황이 옴
* 스프링 부트의 자동 설정 : 서버 시작 시 External Libraries의 모듈마다 있는 spring.factories의 구성 후보들에서 파일에 설정된 클래스는 모두 불러오고, 이후에는 프로젝트에서 사용할 것들만 자동으로 구성해 등록. 즉 빈이 자동으로 등록되고 구성됨
# @SpringBootApplication
스프링 부트 사용에 필요한 기본 설정해줌
```java
package me.siwoli.springbootdeveloper;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
  
@SpringBootApplication  
public class SpringBootDeveloperApplication {  
    public static void main(String[] args) {  
        SpringApplication.run(SpringBootDeveloperApplication.class, args);  
    }  
}
```
SpringApplication.run(애플리케이션의 메인 클래스.class, 커맨드 라인의 인) : 애플리케이션 실행

```java
//SpringBootApplication.java

@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(excludeFilters = { 
@Filter(type = FilterType.CUSTOM, 
		classes = TypeExcludeFilter.class),  
@Filter(type = FilterType.CUSTOM, 
		classes = AutoConfigurationExcludeFilter.class) })  
public @interface SpringBootApplication {  
	...
}
```
## @SpringBootConfiguration 
: 스프링의 @Configuration 상속함. 스프링 부트 관련 설정
## @ComponentScan 
: @Component를 가진 클래스들을 찾아 빈으로 등록함. @Component를 감싸는 annotation들도 포함.

|@Component wrapper annotation| |
|---|---|
|@Configuration|설정 파일 등록|
|@Repository|ORM 매핑
|@Controller, @RestController|라우터
|@Service|비즈니스 로직|

## @EnableAutoConfiguration
: 스프링 부트 서버가 실행될때 스프링 부트의 메타 파일을 읽고 정의된 설정들을 자동으로 구성함. @EnableAutoConfiguration을 사용할때 spring.factories의 클래스들이 모두 자동 설정됨. 즉 *자동 설정으로 등록되는 빈을 읽고 등록함*

# @RestController
: 라우터 역할 -> 이게 있어야 클라이언트의 요청에 맞는 메서드를 실행할 수 있음
```
💡라우터: HTTP요청과 메서드를 연결하는 장치
```
@RestController구현 클래스를 보면
```java
//RestController.java

@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Controller  
@ResponseBody  
public @interface RestController {  
    @AliasFor(annotation = Controller.class)  
    String value() default "";  
}
```
@Controller와 @ResponseBody가 합쳐져 @RestController가 된것 임을 알 수 있음

@Controller 구현 클래스를 보면
```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Component  
public @interface Controller {  
	@AliasFor(annotation = Component.class)  
    String value() default "";  
}
```
@Controller에 @Component가 있으므로 @RestController는 @SpringBootApplication의 @ComponentScan을 통해 빈으로 등록될 수 있음

# spring boot 3 구조
## 프레젠테이션 계층-Controller
- HTTP요청을 받아서 비즈니스 계층으로 전송
- 비즈니스 계층의 응답을 클라이언트로 전송
- 컨트롤러가 프레젠테이션 계층의 역할을 하며 애플리케이션 내에 여러 개의 컨트롤러가 있을 수 있음
```java
@RestController  
public class TestController {  
  
    @Autowired //TestService 빈 주입  
    TestService testService;  
  
    @GetMapping("/test") //testService에서 얻은 멤버 목록을 반환  
    public List<Member> getAllMembers() {  
        List<Member> members = testService.getAllMembers();  
        return members;  
    }  
}
```
## 비즈니스 계층-Service
- 모든 비즈니스 로직 처리
- 퍼시스턴스 계층에서 필요한 DB값 가져오거나 컨트롤러로부터 전달받은 값을 처리해서 DB에 저장
```java
@Service  
public class TestService {  
    @Autowired //MemberRepository 빈 주입  
    MemberRepository memberRepository;  
  
    //MemberRepository에 저장된 멤버 목록을 모두 가져옴  
    public List<Member> getAllMembers() {  
        return memberRepository.findAll();  
    }  
}
```

## 퍼시스턴스 계층-Repository
- 모든 데이터베이스 관련 로직 처리
- 데이터베이스에 접근하는 DAO객체 사용할 수 있음
- Repository인터페이스는 실제 DB의 테이블에 접근해서 DAO와 매핑함
```java
//Member.java

@NoArgsConstructor(access = AccessLevel.PROTECTED)  
@AllArgsConstructor  
@Getter  
@Entity  
public class Member {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "id", updatable = false)  
    private Long id;  
  
    @Column(name = "name", nullable = false)  
    private String name;  
}
```

```java
//MemberRepository.java

@Repository  
public interface MemberRepository extends JpaRepository<Member, Long> {  
}
```
# 스프링 부트 요청-응답 과정
[DispatcherServlet의 Controller위임 과정 참고](https://velog.io/@guswlsapdlf/Springboot-HTTP-Request-Response-%EA%B3%BC%EC%A0%95)
[spring의 HTTP 통신 흐름 과정 참고](https://data-make.tistory.com/714)
1. 클라이언트 요청
2. 스프링 부트의 DispatcherServlet이 URL 분석 후 요청을 처리할 controller 찾음
3. 적합한 controller와 요청이 매치되면 비즈니스 계층과 퍼시스턴스 계층을 통하며 데이터 처리
4. ViewResolver에서 템플릿 엔진으로 HTML을 만들거나 JSON, XML등의 데이터 생성
5. 생성된 HTML 또는 데이터를 응답으로 반환

# 프로젝트의 디렉터리 구성
프로젝트 폴더
│  build.gradle : 빌드 설정(의존성 추가, 플러그인 추가 등)
│  settings.gradle : 빌드할 프로젝트의 정보 설정
└─src
    ├─main : 프로젝트 실행에 필요한 소스 코드나 리소스 파일이 있음
    │  ├─java
    │  │  └─me
    │  │      └─siwoli
    │  │          │  Main.java
    │  │          │  
    │  │          └─springbootdeveloper
    │  │                  SpringBootDeveloperApplication.java
    │  │                  TestController.java 
    │  │                  TestService.java
    │  │                  Member.java
    │  │                  MemberRepository.java
    │  │                  
    │  └─resources
    │      └─static : 정적 파일(css, js)
    │      └─templates : view 관련 파일(html등)
    │      └─application.yml : ==스프링 부트 설정== 
    └─test : 프로젝트의 소스 코드를 테스트할 코드나 리소스 파일이 있음
        ├─java
        └─resources
