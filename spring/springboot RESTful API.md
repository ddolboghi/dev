# REST(Representational State Transfer) API
- 월드 와이드 웹(WWW)처럼 분산 하이퍼미디어 시스템 아키텍처의 한 형식
- 주고받는 자원에 이름을 규정하고 URI에 명시해 HTTP메서드로 해당 자원의 상태를 주고받는 것
- REST API는 REST아키텍처를 따르는 시스템/애플리케이션 인터페이스
# REST 특징
### 유니폼 인터페이스
- 일관된 인터페이스
- 플랫폼 및 기술에 종속되지 않고 타 언어, 플랫폼, 기술 등과 호환해 사용할 수 있음
### 무상태성(stateless)
- 서버에 상태 정보를 따로 보관하거나 관리하지 않음
- 서버가 클라이언트의 요청에 대해 세션이나 쿠키정보를 별도로 보관하지 않아 각 요청을 별도로 처리함
- 서버가 불필요한 정보를 관리하지 않아 비즈니스 로직의 자유도가 높고 설계가 단순함
### 캐시 가능성
- HTTP표준을 그대로 사용하므로 HTTP의 캐싱 기능 적용할 수 있음
- 응답과 요청이 모두 캐싱 가능한지 Cacheable 명시 필요하며 캐싱 가능한 경우 클라이언트에서 캐시에 저장해두고 같은 요청에 대해서는 해당 데이터를 가져다 사용
- 서버의 트랜잭션 부하가 줄어 효율적이며 사용자 입장에서 성능 개선됨
### 레이어 시스템
- REST 서버는 네트워크 상의 여러 계층으로 구성될 수 있지만 서버의 복잡도와 관계없이 클라이언트는 서버와 연결되는 포인트만 알면 됨
### 클라이언트-서버 아키텍처
- REST서버는 API 제공하고, 클라이언트는 사용자 정보를 관리하는 구조로 분리해 설계 → 서로에 대한 의존성 낮춰줌
# REST URI 설계 규칙
- URI의 마지막에는 ‘/’를 포함하지 않음
- 리소스 이름이 길때 밑줄(_) 대신 하이픈(-) 사용
- URL에는 행위(동사)가 아닌 결과(=자원, 명사)를 포함
- 자원의 종류가 컬렉션인지 도큐먼트인지에 따라 단수, 복수를 나눔
- 행위는 HTTP메서드로 표현되야함
- URI는 소문자로 작성
- 파일의 확장자는 URI에 포함하지 않으며 HTTP에서 제공하는 Accept 헤더 사용하기
# 기능 구현 순서
1. Service 메서드 작성
2. 요청에 따라 필요한 경우 엔티티 클래스에 메서드 작성(UPDATE 등)
3. 요청에 따라 필요한 경우 DTO클래스 생성 
	- 클라이언트 요청으로부터 값을 받을때(CREATE, UPDATE) -> *xxxRequest.java*
	- 응답으로 데이터베이스의 값을 줘야할때(READ) -> *xxxResponse.java*
4. Controller 메서드 작성
# @RequestBody
spring annotation
HTTP요청 시 @RequestBody가 붙은 객체에 응답 값이 매핑됨
# ObjectMapper클래스
```java
@Autowired  
protected ObjectMapper objectMapper;
```
자바 객체를 JSON데이터로 변환하는 직렬화(serialization) 또는 반대로 JSON데이터를 자바 객체로 변환하는 역직렬화(deserialization)할때 사용
> [!info]
> 스프링부트는 객체를 JSON으로 바꿔주는 라이브러리가 내장되있어 자동으로 JSON 형식으로 바꿔주기때문에 필요 없음

# @PathVariable
`import org.springframework.web.bind.annotation.*;`
이게 붙은 변수는 URL에서 { }에 있는 값을 변수에 저장함

