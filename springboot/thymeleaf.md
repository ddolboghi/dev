# build.gradle에 의존성 추가하기
```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
}
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
## Model 클래스
`org.springframework.ui.Model`
- 뷰(html)로 값을 넘겨주는 객체로, 컨트롤러와 뷰의 중간 다리 역할
- 따로 생성할 필요 없이 ==메서드의 인자==로 선언하면 스프링이 알아서 만들어줌
- `addAttribute(키, 값)` : 모델 객체에 키-값 저장
# thymeleaf 문법

|표현식| |예시|
|---|---|---|
|`${...}`|변수의 값|
|`#{...}`|속성 파일 값|
|`@{...}`|URL|
|`*{...}`|선택한 변수의 표현식. `th:object`에서 선택한 객체에 필드명으로 접근|
|`th:text`|텍스트를 표현할때 사용|`th:text=${person.name}`
|`th:each`|컬렉션을 반복할때 사용|`th:each="person:${persons}"`
|`th:if`|조건이 true일때만 표시|`th:if="${person.age}>=20"`
|`th:unless`|조건이 false일때만 표시|`th:unless="${person.age}>=20"`
|`th:href`|이동 경로|`th:href="@{/persons/{id}(id=${person.id})}"`
|`th:with`|변수값으로 지정|`th:with="name=${person.name}"`
|`th:object`|선택한 객체로 지정|`th:object="${person}"`
- `${#temporals.format(날짜데이터, 'yyyy-MM-dd HH:mm')}` : 날짜 형식을 yyyy-MM-dd HH:mm으로 포맷팅
- `th:text="|텍스트 ${...}|"` : 속성에 표현식과 함께 텍스트도 넣어서 같이 나오게 함

### CSRF 관련

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
# 뷰로 데이터를 전달하는 DTO
GET 요청으로 글 목록을 반환하기위해 각 article 데이터를 해당 DTO로 변환해서 전달  
```JAVA
package com.siwoli.blog.dto;  
  
import com.siwoli.blog.domain.Article;  
import lombok.Getter;  
  
//뷰에 데이터를 전달할 dto@Getter  
public class ArticleListViewResponse {  
    private final Long id;  
    private final String title;  
    private final String content;  
  
    public ArticleListViewResponse(Article article) {  
        this.id = article.getId();  
        this.title = article.getTitle();  
        this.content = article.getContent();  
    }  
}
```

1. `findAll()`: 이 메서드는 `blogService` 객체에서 호출되며, 일반적으로 모든 게시물을 데이터베이스에서 검색하여 반환합니다. 반환 유형은 일반적으로 게시물의 리스트입니다.
2. `stream()`: Java 8부터 도입된 스트림 API의 일부입니다. 컬렉션 인스턴스에 대해 이 메서드를 호출하면, 그 컬렉션의 요소들을 처리할 수 있는 '스트림'이 생성됩니다. 스트림은 데이터의 연속적인 흐름을 나타내며, 다양한 연산(필터링, 매핑, 집계 등)을 지원합니다.
3. `map()`: 스트림 API의 메서드 중 하나로, 스트림의 각 요소에 함수를 적용하여 그 결과로 구성된 새로운 스트림을 생성합니다. 여기서는 `ArticleResponse::new`라는 메서드 참조가 전달되었으므로, 각 `Article` 객체가 해당 생성자를 통해 `ArticleResponse` 객체로 변환됩니다.
4. `toList()`: Java 16부터 추가된 Collector 인터페이스의 정적 메서드입니다(Java 16 이전에는 `.collect(Collectors.toList())`). 이 메서드는 스트림의 모든 요소를 리스트에 수집합니다.
# JPA Auditing
[참고](https://webcoding-start.tistory.com/53)

## @EnableJpaAuditing
- 엔티티 객체가 생성또는 수정되었을때 자동으로 엔티티 객체의 특정 필드값을 변경
- JPA Auditing이 활성화되어 `@CreatedDate`, `@LastModifiedDate`가 자동으로 업데이트되도록함
- blog프로젝트에서는 애플리케이션 실행 클래스에 적용
## @EntityListeners(AuditingEntityListener.class)
- 적용된 클래스에 Auditing 기능을 포함시킴