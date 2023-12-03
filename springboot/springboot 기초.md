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
# 스프링부트 프로젝트 구조
프로젝트 폴더
│  build.gradle : 프로젝트에 필요한 의존성, 플러그인 작성
│  settings.gradle : 빌드할 프로젝트의 정보 설정
└─src
    ├─main : 프로젝트 실행에 필요한 소스 코드나 리소스 파일이 있음
    │  ├─java
    │      └─패키지
    │          └─프로젝트명Application.java: 프로그램 시작을 담당하는 파일
    │  └─resources
    │      └─static : 정적 파일 저장(css, js, 이미지)
    │      └─templates: HTML 템플릿 파일 저장
    │      └─application.yml(.properties) : 프로젝트의 환경 설정, 데이터베이스 설정
    └─test : 프로젝트의 소스 코드를 테스트할 코드나 리소스 파일이 있음
        ├─java
        └─resources
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
SpringApplication.run(애플리케이션의 메인 클래스.class, 커맨드 라인의 인자) : 애플리케이션 실행

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

# 레이어 아키텍처
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

# build.gradle
## dependencies
- implementation: 해당 라이브러리 설치. 라이브러리가 변경되도 이 라이브러리와 연관된 모든 모듈들을 컴파일하지 않고 직접 관련이 있는 모듈만 컴파일하므로 rebuild 속도가 빠름
- developmentOnly: 개발환경에서만 적용되는 설정. 운영환경에 배포되는 jar, war에는 developmentOnly로 설치된 라이브러리는 제외됨
- runtimeOnly: 해당 라이브러리가 런타임시에만 필요한 경우
- compileOnly: 해당 라이브러리가 컴파일 단계에서만 필요한 경우
- annotationProcessor: 컴파일 단계에서 애너테이션을 분석하고 처리하기 위해 사용

### 스프링부트 도구 설치
```gradle
dependencies {  
    ...
    //서버 자동 재시작을 위한 라이브러리
    developmentOnly 'org.springframework.boot:spring-boot-devtools'  

	//lombok
    implementation 'org.projectlombok:lombok'  
    annotationProcessor 'org.projectlombok:lombok'  
    //lombok의 test 환경  
    //testImplementation('org.projectlombok:lombok')  
    //testAnnotationProcessor('org.projectlombok:lombok')
}
```
- intellij 서버 자동 재시작 설정: file > settings > build,execution,deployment > compiler > build project automatically 체크
- application.properties에 서버 자동 재시작 설정(선택)
```properties
spring.devtools.restart.enabled=true  
spring.devtools.livereload.enabled=true
```
# lombok
## `@RequiredArgsConstructor`
- 클래스의 `final`인스턴스 변수를 받는 생성자가 자동 생성됨
- ==의존성 주입 시 사용됨==
```java
@RequiredArgsConstructor 
@Getter  
public class HelloLombok { 
	private final String hello; 
	private final int lombok; 
	
	public static void main(String[] args) { 
		HelloLombok helloLombok = new HelloLombok("헬로", 5);
		System.out.println(helloLombok.getHello());
		System.out.println(helloLombok.getLombok()); 
	} 
}
```
아래처럼 생성자를 직접 작성한 경우와 동일함
```java
@Getter 
public class HelloLombok { 
	private final String hello; 
	private final int lombok; 
	
	public HelloLombok(String hello, int lombok) { 
		this.hello hello; 
		this.lombok = lombok; 
	}
	
	public static void main(String[] args) { 
			HelloLombok helloLombok = new HelloLombok("헬로", 5);
			System.out.println(helloLombok.getHello());
			System.out.println(helloLombok.getLombok()); 
	} 
}
```
# 컨트롤러
## URL 매핑
- GET 요청: `@GetMapping("<URL>")`
- POST 요청: `@PostMapping("<URL>")`
- 단순 값 보낼때: 메서드에 `@Responsebody`붙이고 `return 메서드 반환 타입 값`
- 템플릿 보낼때: `return "템플릿 이름"`
- 리다이렉트`return "redirect:<URL>"`: 완전히 새로운 URL로 요청됨
- 포워드`return "forword:<URL>"`: 기존 요청 값들이 유지된 상태로 URL 전환
## 값이 변하는 URL 매핑
```java
@GetMapping(value = "/question/detail/{id}")  
public String detail(Model model, @PathVariable("id") Integer id) {  
    return "question_detail";  
}
```
- 매핑 애너테이션에서 사용한 {값}과 `@PathVariable("값")`이 같아야함 
## URL prefix
- 메서드의 매핑 애너테이션에 중복되는 URL 프리픽스가 있는 경우 클래스에 `@RequestMapping(<중복 URL>)`을 추가하고 메서드에 있는 건 생략할 수 있음
- 프리픽스한 컨트롤러는 항상 해당 URL 프리픽스로 시작해야한다는 규칙이 생김 
## HTTP 요청 파라미터 받기
`@RequestParam`: HttpServletRequest의 getParameter와 동일한 기능
### 기본 사용법
`@RequestParam("가져올 데이터의 이름") [데이터 타입] [변수]`
- HTTP 파라미터 이름이 `변수` 이름과 같으면 @RequestParam의 value 생략 가능
- **템플릿에서 보낸 파라미터 이름: html태그의 name 속성 값**
- HTTP 파라미터가 String, int, Integer 등의 단순 타입이면 @RequestParam 생략 가능
- `@RequestParam Map<...> ...`: Map에 key=파라미터, value=파라미터 값 저장됨
- 동일한 이름의 파라미터 값이 2개 이상이면 MultiValueMap 사용
### 필수 파라미터 지정
`@RequestParam(required=[true/false])`: 파라미터에 필수 여부 지정
- false일때 요청 파라미터에 값이 없으면 null 저장되므로 int 대신 Integer 사용해야함
### 파라미터 기본값 설정
`@RequestParam(defaultValue=[기본값])`: 요펑 파라미터 값이 없으면 기본값 지정됨
- required=false일때 defaultValue 지정하면 int 타입 사용가능
### 검증 Form 적용
- `th:object=${Form 객체}`가 있는 템플릿을 요청하는 메서드의 매개변수는 Form객체여야함
> [!NOTE]
> Form객체 처럼 매개변수로 바인딩한 객체는 Model 객체로 전달하지 않아도 템플릿에서 사용 가능함

# 서비스
- 비즈니스 로직 처리
- 서비스는 컨트롤러와 레포리토리의 중간자로서 엔티티 객체와 DTO객체를 서로 변환해 양방향에 전달하는 역할
- 스프링 부트는 `@Service`(`import org.springframework.stereotype.Service`)가 붙은 클래스를 서비스로 인식함
