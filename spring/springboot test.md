# H2 데이터베이스 사용시 테스트
- H2 데이터베이스는 파일 기반의 데이터베이스이므로 테스트하려면 로컬 서버 실행을 중지해야한다. 
# given-when-then 패턴
given : 테스트 실행 준비; 데이터 생성
when : 테스트 진행; 데이터 저장
then : 테스트 결과 검증; 생성한 데이터와 저장한 데이터 비교

*예시 코드
```java
@DisplayName("새로운 메뉴를 저장한다.")
@Test
public void saveMenuTest() {
	//given: 메뉴를 저장하기 위한 준비
	final String name = "아메리카노";
	final int price = 1100;
	
	final Menu americano = new Menu(name, price);

	//when: 실제로 메뉴를 저장
	final long savedId = menuService.save(americano);

	//then: 메뉴가 잘 추가되었는지 검증
	final Menu savedMenu = menuService.findById(savedId).get();
	assertThat(savedMenu.getName())isEqualTo(name);
	assertThat(savedMenu.getPrice())isEqualTo(price);
}
```

# spring-boot-starter-test의 도구들
- ==JUnit== : 자바 용 단위 테스트 프레임워크
- Spring Test & Spring Boot Test : 스프링 부트 애플리케이션을 위한 통합 테스트 지원
- ==AssertJ== : 검증문 작성하는데 사용되는 라이브러리
- Hamcrest : 표현식을 보다 이해하기 쉽게 만드는데 사용되는 Matcher라이브러리
- Mockito : 테스트에 사용할 가짜 객체인 Mock객체를 쉽게 만들고, 관리 및 검증할 수 있는 테스트 프레임워크
- JSONassert : JSON용 검증문 라이브러리
- JsonPath : JSON 데이터에서 특정 데이터를 선택하고 검색하기 위한 라이브러리

# JUnit & AssertJ
- 테스트 방식을 구분할 수 있는 annotation제공
- @Test로 메서드를 호출할때마다 새 인스턴스 생성해서 테스트끼리 영향 안줌 -> 독립 테스트 가능
- 예상 결과를 검증하는 assertion메서드 제공
- 단순한 사용법 -> 테스트 코드 작성 시간 단축
- 자동 실행, 자체 결과를 확인하고 즉각적인 피드백 제공

| | | |
|---|---|---|
|@Test|테스트 수행하는 메서드 정의||
|@BeforeAll|전체 테스트를 시작하기 전에 한 번만 호출되는 *static*메서드 정의|데이터베이스 연결, 테스트 환경 초기화|
|@BeforeEach|각 테스트 메서드가 실행되기 전에 동작하는 메서드 정의|테스트 메서드에서 사용하는 객체 초기화, 테스트에 필요한 값 미리 넣음
|@AfterAll|전체 테스트 마치고 종료하기 전에 한 번만 호출되는 *static*메서드 정의|데이터베이스 연결 종료, 공통 자원 해제|
|@AfterEach|각 테스트 메서드가 종료되면서 호출되는 메서드 정의|테스트 이후 특정 데이터 삭제|
|@Disabled|이게 지정된 테스트는 실행 안하고 여기에 딸려오는 @BeforeEach, @AfterEach도 실행 안함 대신 해당 테스트 메서드가 비활성화됐다는 로그 출력|
`Assertions.assertEquals(기대값, 실제값);`

AssertJ의 검증 클래스 : `assertThat(a+b).isEqualTo(sum);`
검증단계에서 AssertJ를 적용하면 코드의 가독성이 좋아짐

|`assertThat()`의 메서드| |
|---|---|
|`isEqualTo(A)`|A값과 같은지|
|`isNotEqualTo(A)`|A값과 다른지
|`contains(A)`|A값을 포함하는지
|`doesNotContain(A)`|A값을 포함하지 않는지
|`startsWith(A)`|접두사가 A인지
|`endsWith(A)`|접미사가 A인지
|`isEmpty()`|비어있는 값인지
|`isNotEmpty()`|비어있지 않은 값인지
|`isPositive()`|양수인지
|`isNegative()`|음수인지
|`isGreaterThan(1)`|1보다 큰 값인지
|`isLessThan(1)`|1보다 작은 값인지
|`isZero()`|값이 0인지

# controller test
```java
@SpringBootTest//테스트용 애플리케이션 컨텍스트 생성  
@AutoConfigureMockMvc//MockMvc 생성  
class TestControllerTest {  
  
    @Autowired  
    protected MockMvc mockMvc;  
  
    @Autowired  
    private WebApplicationContext context;  
  
    @Autowired  
    private MemberRepository memberRepository;  
  
    @BeforeEach //MockMvc설정
    public void mockMvcSetUp() {  
        this.mockMvc = MockMvcBuilders.webAppContextSetup(context).build();  
    }  
  
    @AfterEach //테이블의 모든 데이터 삭제
    public void cleanUp() {  
        memberRepository.deleteAll();  
    }  

	@DisplayName("getAllMembers: 아티클 조회에 성공한다.")  
	@Test  
	public void getAllMembersTest() throws Exception {  
	    //given: 멤버 저장
	    final String url = "/test";  
	    Member savedMember = memberRepository.save(new Member(1L, "시월이"));  
	  
	    //when: 멤버 리스트를 조회하는 API 호출
	    final ResultActions result = mockMvc.perform(get(url) //(1)  
	            .accept(MediaType.APPLICATION_JSON)); //(2)  
	  
	    //then: 응답 코드가 200이면 Ok이고, 반환받은 값 중에 0번째 값 검증
	    result  
	            .andExpect(status().isOk()) //(3)  
	            //(4)       
	            .andExpect(jsonPath("$[0].id").value(savedMember.getId()))  
			.andExpect(jsonPath("$[0].name").value(savedMember.getName()));  
	}
}
```
`@SpringBootTest`
- @SpringBootApplication이 있는 메인 애플리케이션 클래스를 찾고 그 클래스에 포함된 빈을 찾은 다음, 테스트용 애플리케이션에 컨텍스트를 만듬

`@AutoConfigureMockMvc`
- MockMvc를 생성하고 자동으로 구성
- MockMvc는 애플리케이션을 서버에 배포하지 않고도 테스트용 MVC환경을 만들어 요청, 전송, 응답 기능을 제공하는 유틸리티 클래스로, 컨트롤러를 테스트할때 사용됨

(1) `perform()` : 요청을 전송하는 메서드. ResultActions객체 반환
(2) `.accept()` : 요청을 보낼때 무슨 타입으로 응답을 받을지 결정하는 메서드
(3) `.andExpect()` : 응답을 검증하는 ResultActions객체의 메서드. TestController에서 만든 API는 응답으로 OK(200)을 반환하므로 `status().isOk()`로 응답코드가 200인지 확인
(4) `jsonPath("$[몇번째데이터(인덱스)].${필드명}")` : JSON 응답값을 가져오는 메서드

|HTTP 주요 응답 코드|매핑 메서드|
|---|---|
|200 OK|`isOk()`|
|201 Created|`isCreated()`|
|400 Bad Request|`isBadRequest()`|
|403 Forbidden|`isForbidden()`|
|404 Not Found|`isNotFound()`|
|400번대|`is4xxClientError()`|
|500 Internal Server Error|`isInternalServerError()`|
|500번대|`is5xxServerError()`|

테스트 코드는 controller뿐만아니라 service도 작성하는게 좋음

# Tip
## `@Rollback`
- 테스트 코드에서 insert, update, delete한 데이터들을 다시 rollback 시켜줌
- `value = false` 지정시 rollback 안함(`@Commit`과 같음)
## `@AutoConfigureTestDatabase`
[참고](https://0soo.tistory.com/41)