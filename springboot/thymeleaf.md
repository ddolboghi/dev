# build.gradle에 의존성 추가하기
```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect'
}
```
# thymeleaf-layout-dialect
: header나 footer같이 공통으로 쓰이는 부분 설정
```html
xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
```
- 해당 파일을 기본 레이아웃으로 선언 
```html
<!-- 헤더 불러오는 부분 -->
<div th:replace="fragments/header :: header"></div>

<!-- 해당 부분의 content 작성하는 부분 -->
<div layout:fragment="content"></div> 

<!-- 푸터 불러오는 부분 -->
<div th:replace="fragments/footer :: footer"></div>
```

# view로 값을 넘겨주는 controller
- Controller클래스에 `@Controller`를 붙이면 메서드 리턴값은 뷰의 이름이 됨
- resource/templates/뷰 이름.html로 값을 넘겨줌
```java
import org.springframework.ui.Model;

@Controller
public class ExampleController {  
    @GetMapping("/thymeleaf/example")  
    public String thymeleafExample(Model model) {  
        Person examplePerson = new Person();  
        examplePerson.setId(1L);  
        examplePerson.setName("siwoli");  
        examplePerson.setAge(4);  
        examplePerson.setHobbies(List.of("운동", "독서"));  

		//"person"이라는 키로 Person객체를 모델 객체에 저장  
        model.addAttribute("person", examplePerson);
        model.addAttribute("today", LocalDate.now());  
          
        return "example";//example.html라는 뷰 조회  
    }
```
## Model 객체
`org.springframework.ui.Model`
- 컨트롤러와 뷰의 중간 다리 역할로, 뷰(html)에 값을 넘겨줌
- 따로 생성할 필요 없이 ==메서드의 인자==로 선언하면 스프링이 알아서 만들어줌
- `addAttribute(키, 값)` : 모델 객체에 키-값 저장
# thymeleaf 문법
- html 태그에서 `th:`로 시작하는 속성이 타임리프 템플릿 엔진이 사용하는 속성으로, 자바 코드와 연결됨

| 속성               |                                                                                                                                                      | 예시                                   |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| `${...}`           | 자바 객체의 값                                                                                                                                       |                                        |
| `#{...}`           | 속성 파일 값                                                                                                                                         |                                        |
| `@{...}`           | URL                                                                                                                                                  |                                        |
| `*{...}`           | 선택한 변수의 표현식. `th:object`에서 선택한 객체에 필드명으로 접근                                                                                  |                                        |
| `th:text`          | 값을 텍스트로 출력                                                                                                                                   | `th:text=${person.name}`               |
| `<...>[[ ]]</...>` | 대괄호를 사용해 값을 텍스트로 직접 출력                                                                                                              | `<td>[[${person.name}]]</td>`          |
| `th:each`          | 컬렉션을 반복할때 사용                                                                                                                               | `th:each="person:${persons}"`          |
| `th:if`            | 조건이 true일때만 표시                                                                                                                               | `th:if="${person.age}>=20"`            |
| `th:unless`        | 조건이 false일때만 표시                                                                                                                              | `th:unless="${person.age}>=20"`        |
| `th:href`          | 이동 경로                                                                                                                                            | `th:href="@{|/persons/${person.id}|}"` |
| `th:with`          | 변수값으로 지정                                                                                                                                      | `th:with="name=${person.name}"`        |
| `th:object`        | 선택한 객체로 지정                                                                                                                                   | `th:object="${person}"`                |
| `th:field`         | `th:object`로 지정한 객체의 필드에 접근. `*{필드명}`사용. 해당 html태그의 id, name, value를 자동 생성하고 타임리프가 value속성에 기존 값을 채워 넣음 | `th:field=*{content}`                  |
| `th:replace`|공통 템플릿을 템플릿 내에 삽입|`th:replace="~{공통 템플릿 이름 :: 공통 템플릿의 th:fragment 속성명}"`|
- `${#temporals.format(날짜데이터, 'yyyy-MM-dd HH:mm')}` : 날짜 형식을 yyyy-MM-dd HH:mm으로 포맷팅
- `#lists.size(이터러블객체)`: **이터러블객체의 길이 반환**하는 타임리프 유틸리티
- 타임리프는 문자열을 연결할때 `|`사용
- `{|문자열 ${...}|}`: 문자열과 자바 객체의 값을 더할때 반드시 `|`로 감싸야함
- `th:each="개별 객체, loop : ${반복 객체}"`
	- loop.index: 반복 순서, 0부터 1씩 증가
	- loop.count: 반복 순서, 1부터 1씩 증가
	- loop.size: 반복 객체의 요소 개수
	- loop.first: 루프의 첫번째 순서인 경우 true
	- loop.last: 루프의 마지막 순서인 경우 true
	- loop.odd: 루프의 홀수번째 순서인 경우 true
	- loop.even: 루프의 짝수번째 순서인 경우 true
	- loop.current: 현재 대입된 객체 = 개별 객체
- `sec:authorize`: 로그인 여부를 알려주는 속성
	- `sec:authorize="isAnonymous()"`: 로그인되지 않은 경우에만 해당 html요소 표시
	- `sec:authorize="isAuthenticated()"`: 로그인된 경우에만 해당 html요소 표시

### form태그의 CSRF 방지

```html
<form action="/members/new" role="form" method="post" th:object="${memberFormDto}">
		...
		<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />
</form>
```

위와 같이 기본 action속성을 사용하면 직접 input hidden CSRF토큰을 매개변수로 갖는 코드를 작성해야하지만

```html
<form th:action="@{/members/new}" role="form" method="post" th:object="${memberFormDto}">
		...
</form>
```
th:action을 사용하면 자동으로 input hidden CSRF 토큰이 생성됨
# 뷰 작성하기
- html에 `<html xmlns:th="http://www.thymeleaf.org">`작성해야 thymeleaf 사용할 수 있음
```html
<p th:text="${#temporals.format(today, 'yyyy-MM-dd')}"></p>  
<div th:object="${person}">  
    <p th:text="|이름: *{name}|"></p>  
    <p th:text="|나이: *{age}|"></p>  
    <p>취미</p>  
    <ul th:each="hobby: *{hobbies}">  
        <li th:text="${hobby}"></li>  
        <span th:if="${hobby=='운동'}">(대표 취미)</span>  
    </ul></div>  
<a th:href="@{/api/articles/{id}(id=${person.id})}">글 보기</a>
```
- `th:object`로 모델에서 받은 객체 중 "person"이라는 키를 가진 객체의 데이터를 하위 태그에 지정
- 하위 태그에서는 `*{필드명}`로 상위 태그에 적용한 객체의 필드값에 접근
- `th:each="hobby: *{hobbies}"`는 hobbies배열의 반복자를 hobby로 지정하고 하위 태그에서 `${hobby}`로 배열의 원소에 접근
# CSS 적용하기
```css
<link rel="stylesheet" type="text/css" th:href="@{/style.css}">
```
- th:href="@{static디렉터리에 있는 css파일}"
- static디렉터리가 루트 디렉터리이므로 그 밑의 경로를 지정해야함
# 오류 메시지 출력
- Form 검증이 실패한 경우 타임리프에서 오류 메시지 출력하는 법
1. `th:object="${Form객체}"`: 폼의 속성들이 해당 폼 객체의 속성들로 구성된다는 것을 타임리프 엔진에 알려줌. 오류 메시지를 표시할 태그를 포함하는 가장 바깥의 태그에 추가
2. `#fields.hasAnyErrors()`: 검증이 실패한 경우 true 반환. 메시지를 출력할 바로 상위 태그에 추가
3. `th:each="err : ${#fields.allErrors()}" th:text="${err}"`: 검증 실패한 모든 오류 메시지 출력
4. 같은 `th:object`가 적용된 템플릿을 요청하는 요청 메서드의 매개변수도 Form객체로 수정