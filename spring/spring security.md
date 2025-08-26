: 스프링 기반의 애플리케이션 인증, 인가, 권한을 담당하는 스프링 하위 프레임워크
- CSRF공격 방어, 세션 고정(session fixation)공격 방어, 요청 헤더를 보안 처리해줌
- 필터 기반으로 동작
- 세션 기반 인증 사용 : 사용자마다 사용자의 정보를 담은 세션을 생성하고 저장해서 인증
> [!NOTE]
> Authenticate: 로그인
> Authorize: 인증된 사용자가 어떤 것을 할 수 있는지(권한)
# SecurityFilterChain
SecurityContext : 접근 주체와 인증에 대한 정보를 담고 있는 객체

|필터명| |
|---|---|
|SecurityContextPersistenceFilter|SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 역할
|LogoutFilter|설정된 로그아웃 URL로 오는 요청을 확인해 해당 사용자를 로그아웃 처리|
|UsernamePasswordAuthenicationFilter|인증 관리자. 폼 기반 로그인을 할때 사용되는 필터로 아이디, 패스워드 데이터를 파싱해 인증 요청을 위임함. 인증이 성공하면 AuthenticationSuccessHandler를, 인증에 실패하면 AuthenticationFailureHandler 실행|
|DefaultloginPageGeneratingFilter|사용자가 로그인 페이지를 따로 지정하지 않았을때 기본으로 설정하는 로그인 페이지 관련 필터|
|BasicAuthenticationFilter|요청 헤더에 있는 아이디와 패스워드를 파싱해서 인증요청을 위임함. 인증이 성공하면 AuthenticationSuccessHandler를, 인증에 실패하면 AuthenticationFailureHandler를 실행|
|(RememberMeAuthenticationFilter)->RequestCacheAwareFilter|로그인 성공 후, 관련있는 캐시 요청이 있는지 확인하고 캐시 요청 처리. e.g.로그인하지 않은 상태로 방문했던 페이지를 기억해뒀다가 로그인 이후에 그 페이지로 이동시켜줌|
|SecutiryContextHolderAwareRequestFilter|HttpServletRequest정보를 감쌈. 필터 체인 상의 다음 필터들에게 부가 정보 제공
|AnonymousAuthenticationFilter|이 필터가 호출될때까지 인증되지 않았다면 익명 사용자 전용 객체인 AnonymousAuthentication을 만들어 SecurityContext에 넘겨줌|
|SessionManagementFilter|인증된 사용자와 관련된 세션 관련 작업 진행. e.g.세션 변조 방지 전략 설정, 유효하지 않은 세션 처리, 세션 생성 전략 설정 등|
|ExceprionTranslationFilter|요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달|
|FilterSecurityInterceptor|인가 관련 설정을 하는 접근 결정 관리자. AccessDecisionManager로 권한 부여 처리를 위임해 접근 제어 결정을 쉽게 해줌. 이 과정에서는 이미 사용자가 인증되어 있으므로 유효한 사용자인지도 알 수 있음.
![[SecurityFilterChain.png]]
## 아이디와 패스워드 기반 폼 로그인 인증 절차
1. 사용자가 폼에 아이디와 패스워드를 입력하면 HTTPServletRequest에 아이디와 비밀번호 정보가 전달됨. 이때 AuthenticationFilter가 넘어온 아이디와 비밀번호의 유효성 검사를 함
2. 유효성 검사가 끝나면 실제 구현체인 UsenamePasswordAuthenticationToken을 만들어 넘겨줌
3. 전달받은 인증용 객체인 Token을 AuthenticationManager에 보냄
4. Token을 AuthenticationProvider에 보냄
5. 사용자 아이디를 UserDetailService에 보냄. UserDetailService는 사용자 아이디로 찾은 사용자의 정보를 UserDetails객체로 만들어 AuthenticationProvider에 전달
6. DB에 있는 사용자 정보를 가져옴
7. 입력 정보와 UserDetails의 정보를 비교해 실제 인증 처리
8~10까지 인증이 완료되면 SecutiryContextHolder에 Authentication을 저장
인증 성공 여부에 따라 성공하면 AuthenticationSuccessHandler, 실패하면 AuthenticationFailureHandler를 실행
![[spring secutiry - 폼 로그인.canvas|spring secutiry - 폼 로그인]]
# spring boot에서 인증, 인가 기능 구현
## 의존성 추가
```gradle
implementation 'org.springframework.boot:spring-boot-starter-security'  
//thymeleaf에서 spring security를 사용하기 위한 의존성 추가  
implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'  
testImplementation 'org.springframework.security:spring-security-test'
```

## 시큐리티 설정 파일 작성하기
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    @Bean  
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
        http  
                .authorizeHttpRequests((authorizeHttpRequests) -> authorizeHttpRequests  
                        .requestMatchers(new AntPathRequestMatcher("/**")).permitAll());  
        return http.build();  
    }  
}
```
- `@Configuration`: 스프링의 환경설정 파일임을 의미하는 애너테이션
- `@EnableWebSecurity`: 모든 요청 URL이 스프링 시큐리티의 제어를 받도록 만드는 애너테이션
- 스프링 시큐리티의 세부 설정은 SecurityFilterChain 빈을 생성하여 설정
- `invalidateHttpSession(true)` : 로그아웃 이후에 세션을 전체 삭제할지 여부 설정
# CSRF(cross site request forgery) 오류 해결
- 스프링 시큐리티가 CSRF 토큰 값을 세션을 통해 발생하고 웹 페이지에서는 폼 전송시에 해당 토큰을 함께 전송해 실제 웹 페이지에서 작성된 데이터가 전달되는지 검증
- H2 콘솔은 CSRF 토큰을 발행하는 기능이 없어 403오류 발생
  --> 스프링 시큐리티가 CSRF 처리시 H2 콘솔은 예외로 처리하도록 시큐리티 설정 파일 수정해야함
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    @Bean  
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
        http  
		        ...
                
                .csrf((csrf) -> csrf  
        .ignoringRequestMatchers(new AntPathRequestMatcher("/h2-console/**")));
        
        return http.build();  
    }  
}
```
# clickjacking 공격 오류 해결
- H2 콘솔의 화면은 frame구조로 작성됨
- 스프링 시큐리티는 사이트의 콘텐츠가 다른 사이트에 포함되지 않도록 하기 위해 `X-Frame-Options`를 사용해 clickjacking 공격 방지
- `X-Frame-Options`의 헤더 값을 SAMEORIGIN으로 설정하면 frame에 포함된 페이지가 페이지를 제공하는 사이트와 동일한 경우에는 계속 사용 가능
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    @Bean  
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
        http 
		        ...
		        
                .headers((headers) -> headers  
                        .addHeaderWriter(new XFrameOptionsHeaderWriter(  
                                XFrameOptionsMode.SAMEORIGIN  
                        )));  
        return http.build();  
    }  
}
```

# 회원가입 기능 구현
## 1. 회원 정보를 위한 엔티티 생성
- 엔티티는 ==UserDetails인터페이스==를 상속하거나 기본적인 엔티티의 형태로 생
- 필수 엔티티 속성: username, password, email

|UserDetails의 메서드|반환 타입| |
|---|---|---|
|`getAuthorities()`|`Collection<? extends GrantedAuthority`|사용자가 가지고 있는 권한의 목록 반환
|`getUsername()`|`String`|사용자를 식별할 수 있는 사용자 이름 반환. 이때 사용되는 사용자 이름은 반드시 `unique=true`여야함(고유해야함)
|`getPassword()`|`String`|사용자의 비밀번호 반환. 이때 비밀번호는 암호화해서 저장해야함
|`isAccountNonExpired()`|`boolean`|계정이 만료되었는지 확인하는 메서드. 만료 안되었으면 true반환|
|`isAccountNonLocked()`|`boolean`|계정이 잠겼는지 확인하는 메서드. 안 잠겨있으면 true반환|
|`isCredentialsNonExpired()`|boolean|비밀번호가 만료되었는지 확인하는 메서드. 만료 안되었으면 true반환|
|`isEnabled()`|`boolean`|계정이 사용 가능한지 확인하는 메서드. 사용가능하면 true반환|

## 2. 레포지터리와 서비스
### 레포지터리
```java
public interface UserRepository extends JpaRepository<SiteUser, Long> {  
}
```
### 비밀번호 암호화
- `BCryptPasswordEncoder`로 암호화해 저장
- PasswordEncoder 빈을 등록해서 `BCryptPasswordEncoder`사용하면 나중에 암호화 방식이 바껴도 수정하기 쉬움
- SecurityConfig에서 PasswordEncoder 빈 등록하기
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    ...
    
    @Bean  
    PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```

### 서비스
```java
@RequiredArgsConstructor  
@Service  
public class UserService {  
    private final UserRepository userRepository;  
    private final PasswordEncoder passwordEncoder;  
  
    public SiteUser create(String username, String email, String password) {  
        SiteUser user = new SiteUser();  
        user.setUsername(username);  
        user.setEmail(email);  
        user.setPassword(passwordEncoder.encode(password));  
        this.userRepository.save(user);  
        return user;  
    }  
}
```

## 3. 회원가입 폼
```java
@Getter  
@Setter  
public class UserCreateForm {  
    @Size(min = 3, max = 25)  
    @NotEmpty(message = "아이디는 필수항목입니다.")  
    private String username;  
  
    @NotEmpty(message = "비밀번호는 필수항목입니다.")  
    private String password;  
  
    @NotEmpty(message = "비밀번호 확인은 필수항목입니다.")  
    private String passwordCheck;  
  
    @NotEmpty(message = "이메일은 필수항목입니다.")  
    @Email  
    private String email;  
}
```

## 4. 회원가입 컨트롤러
```java
@RequiredArgsConstructor  
@Controller  
@RequestMapping("/user")  
public class UserController {  
    private final UserService userService;  
      
    @GetMapping("/signup")  
    public String signup(UserCreateForm userCreateForm) {  
        return "signup_form";  
    }  
  
    @PostMapping("/signup")  
    public String signup(@Valid UserCreateForm userCreateForm, BindingResult bindingResult) {  
        if (bindingResult.hasErrors()) {  
            return "signup_form";  
        }  
  
        if (!userCreateForm.getPassword().equals(userCreateForm.getPasswordCheck())) {  
            bindingResult.rejectValue("passwordCheck", "passwordInCorrect","비밀번호가 일치하지 않습니다.");  
            return "signup_form";  
        }  
        try {  
            userService.create(userCreateForm.getUsername(), userCreateForm.getEmail(), userCreateForm.getPassword());  
        } catch (DataIntegrityViolationException e) {  
            e.printStackTrace();  
            bindingResult.reject("signupFailed", "이미 등록된 회원입니다.");  
            return "signup_form";  
        } catch (Exception e) {  
            e.printStackTrace();  
            bindingResult.reject("signupFailed", e.getMessage());  
        }  
  
        return "redirect:/";  
    }  
}
```
1. 먼저 GET요청에는 회원가입 템플릿을 보냄
2. 회원가입 템플릿에 입력한 데이터를 UserCreateForm으로 보내고 검증
3. try-catch로 중복 회원가입 처리하고 userService로 회원 데이터 저장
4. 메인페이지로 리다이렉트

- `bindingResult.rejectValue(오류 발생한 필드, 오류코드, 오류메시지)`
- `bindingResult.reject(오류코드, 오류메시지):` 특정 필드의 오류가 아닌 일반적인 오류를 등록할때 사용

## 5. 회원가입 템플릿
```java
<html lang="ko"  
      xmlns:th="http://www.thymeleaf.org"  
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"  
      layout:decorate="~{layout}">  
<div layout:fragment="content">  
    <div>
	    <div>            
		    <h4>회원가입</h4>  
        </div>    
    </div>    
	<form th:action="@{/user/signup}" th:object="${userCreateForm}" method="post">  
	    <div th:replace="~{form_errors :: formErrorsFragment}"></div>  
		<div>            
			<label for="username">아이디</label>  
			<input type="text" th:field="*{username}">  
		</div>       
		<div>            
			<label for="password">비밀번호</label>  
	        <input type="password" th:field="*{password}">  
	    </div>        
	    <div>            
		    <label for="password-check">비밀번호 확인</label>  
	        <input type="password" th:field="*{passwordCheck}">  
	    </div>        
		<div>
			<label for="email">이메일</label>  
	        <input type="email" th:field="*{email}">  
	    </div>
	    <button type="submit">회원가입</button>  
    </form>
</div>  
</html>
```
- th:object로 userCreateForm객체를 생성

# 로그인/로그아웃 기능 구현
## 1. SecurityFilterChain에 로그인 URL 등록
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    @Bean  
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
        http  
		        ...
                .formLogin((formLogin) -> formLogin  
                        .loginPage("/user/login")  
                        .defaultSuccessUrl("/"));  
        return http.build();
```

## 2. 컨트롤러
```java
@RequiredArgsConstructor  
@Controller  
@RequestMapping("/user")  
public class UserController {  
    ...
  
    @GetMapping("/login")  
    public String login() {  
        return "login_form";  
    }  
}
```
- GET방식의 로그인 메서드 추가
- 실제 로그인을 진행하는 POST방식의 메서드는 스프링 시큐리티가 대신 처리하므로 직접 구현할 필요 없음

## 3. 템플릿
```html
<html lang="ko"  
      xmlns:th="http://www.thymeleaf.org"  
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"  
      layout:decorate="~{layout}">  
<div layout:fragment="content">  
    <form th:action="@{/user/login}" method="post">  
        <div th:if="${param.error}">  
            <div class="err">아이디 또는 비밀번호를 확인해주세요.</div>  
        </div>        
        <div>            
	        <label for="username">아이디</label>  
            <input type="text" name="username" id="username">  
        </div>        
        <div>            
	        <label for="password">비밀번호</label>  
            <input type="password" name="password" id="password">  
        </div>        
        <button type="submit">로그인</button>  
    </form>
</div>  
</html>
```
- 로그인 실패시 리다이렉트되는 페이지의 파라미터로 error가 전달됨
- `th:if="${param.error}"`: 파라미터로 error가 전달되면(로그인이 실패하면)

## 4. 레포지토리
- 사용자 이름으로 사용자를 찾는 메서드 추가
```java
public interface UserRepository extends JpaRepository<SiteUser, Long> {  
    Optional<SiteUser> findByUsername(String username);  
}
```

## 5. UserRole
- 인증 후에 사용자에게 부여할 권한 필요
- enum을 이용해 권한 목록 작성
```java
@Getter  
public enum UserRole {  
    ADMIN("ROLE_ADMIN"),  
    USER("ROLE_USER");  
  
    private final String value;  
  
    UserRole(String value) {  
        this.value = value;  
    }  
}
```

## 6. UserDetailService의 구현 서비스 작성
- 회원가입으로 데이터베이스에 저장된 회원 정보를 조회하는 기능 필요
```java
@RequiredArgsConstructor  
@Service  
public class UserSecurityService implements UserDetailsService {  
    private final UserRepository userRepository;  
    private final String ADMIN_NAME = "admin";  
  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
        Optional<SiteUser> _siteUser = this.userRepository.findByUsername(username);  
        if (_siteUser.isEmpty()) {  
            throw new UsernameNotFoundException("사용자를 찾을수 없습니다.");  
        }  
        SiteUser siteUser = _siteUser.get();  
        List<GrantedAuthority> authorities = new ArrayList<>();  
        authorities.add(grantAuthority(username));  
        return new User(siteUser.getUsername(), siteUser.getPassword(), authorities);  
    }  
  
    private SimpleGrantedAuthority grantAuthority(String username) {  
        if (ADMIN_NAME.equals(username)) {  
            return new SimpleGrantedAuthority(UserRole.ADMIN.getValue());  
        }  
        return new SimpleGrantedAuthority(UserRole.USER.getValue());  
    }  
}
```
- 스프링 시큐리티가 제공하는 UserDetailService인터페이스는 `loadUserByUsername`메서드를 구현하도록 강제함
- 사용자명, 비밀번호, 권한을 입력으로 스프링 시큐리티의 User객체를 생성해 리턴
- 스프링 시큐리티는 `loadUserByUsername`메서드에 의해 리턴된 User객체의 비밀번호가 화면으로부터 입력 받은 비밀번호와 일치하는지 검사하는 로직을 내부적으로 가지고 있음

## 7. SecurityConfig에 AuthenticationManager 빈 생성하기
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
	@Bean  
    AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {  
        return authenticationConfiguration.getAuthenticationManager();  
    }  
}
```
- AuthenticationManager는 스프링 시큐리티의 인증을 담당함
- AuthenticationManager는 사용자 인증시 UserDetailService를 구현한 서비스 클래스와 PasswordEncoder클래스 사용함

## 8. 로그아웃 구현
```java
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    @Bean  
    SecurityFilterChain filterChain(HttpSecurity http) throws Exception {  
        http  
                ...
                .formLogin((formLogin) -> formLogin  
                        .loginPage("/user/login")  
                        .defaultSuccessUrl("/"))  
                .logout((logout) -> logout  
                        .logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))  
                        .logoutSuccessUrl("/")  
                        .invalidateHttpSession(true));  
        return http.build();  
    }  
}
```
- SecurityFilterChain의 http 체인메서드로 logout추가
- `invalidateHttpSession(true)`: 로그아웃시 생성된 사용자 세션 삭제

## 9. 템플릿에 컨트롤러의 로그인 링크 추가
```html
<li>  
    <a sec:authorize="isAnonymous()" th:href="@{/user/login}">로그인</a>  
    <a sec:authorize="isAuthenticated()" th:href="@{/user/logout}">로그아웃</a>  
</li>
```

# 글쓴이 추가하기
1. 글쓴이(사용자)가 있어야하는 엔티티에 `@ManyToOne`으로 사용자 클래스 속성 추가
```java
@Getter  
@Setter 
@Entity  
public class Answer {  
  
    ...
    
    @ManyToOne  
    private SiteUser author;  
}
```

2. 사용자 서비스 클래스에 사용자 조회 메서드 추가
```java
@RequiredArgsConstructor  
@Service  
public class UserService {  
	...
  
    public SiteUser getUser(String username) {  
        Optional<SiteUser> siteUser = this.userRepository.findByUsername(username);  
        if (siteUser.isEmpty()) {  
            throw new DataNotFoundException("siteuser not found");  
        }  
        return siteUser.get();  
    }  
}
```

3. 사용자가 데이터를 생성하는 행위가 있는 다른 서비스의 메서드에서 해당 사용자를 저장하도록 수정
```java
@RequiredArgsConstructor  
@Service  
public class AnswerService {  
    private final AnswerRepository answerRepository;  
  
    public void create(Question question, String content, SiteUser siteUser) {  
        Answer answer = new Answer();  
        answer.setContent(content);  
        answer.setCreateDate(LocalDateTime.now());  
        answer.setQuestion(question);  
        answer.setAuthor(siteUser);  
        this.answerRepository.save(answer);  
    }  
}
```

4. 컨트롤러에서 사용자 생성 요청(create 등) 메서드에 매개변수로 Principal 객체 지정
```java
@RequestMapping("/answer")  
@RequiredArgsConstructor  
@Controller  
public class AnswerController {  
    private final QuestionService questionService;  
    private final AnswerService answerService;
    private final UserService userService;

	@PreAuthorize("isAuthenticated()")
    @PostMapping("/create/{id}")  
	public String createAnswer(  
        Model model,  
        @PathVariable("id") Integer id,  
        @Valid AnswerForm answerForm,  
        BindingResult bindingResult,  
        Principal principal) {  
	    Question question = this.questionService.getQuestion(id);  
	    SiteUser siteUser = this.userService.getUser(principal.getName());  
	  
	    if (bindingResult.hasErrors()) {  
	        model.addAttribute("question", question);  
	        return "question_detail";  
	    }  
	    this.answerService.create(question, answerForm.getContent(), siteUser);  
	    return String.format("redirect:/question/detail/%d", id);  
	}
}
```
## Principal 객체
- 스프링 시큐리티가 제공하는 Principal 객체는 현재 로그인한 사용자에 대한 정보를 가짐
- ==Prinripal객체는 로그인을 해야만 생성되므로 로그아웃 상태에서 Principal객체는 null임==
- `Principal객체.getName()`: 현재 로그인한 사용자의 사용자명(사용자ID) 리턴
## `@PreAuthorize("isAuthenticated()")`
- 로그인이 필요한 컨트롤러의 메서드에 붙임
- 로그아웃 상태에서 해당 메서드가 호출되면 로그인 페이지로 이동됨
- SecurityConfig 클래스에 ==`@EnableMethodSecurity(prePostEnabled = true(default))`==를 붙여 `@PreAuthorize`가 동작할 수 있도록해야함
- 로그인하지 않은 상태에서 `@PreAuthorize`가 붙은 컨트롤러 메서드(요청)이 실행되면 로그인 페이지로 이동하고, 로그인하면 스프링 시큐리티에 의해 다시 해당 메서드를 실행하기위한 페이지로 리다이렉트된다.
```java
@Configuration  
@EnableWebSecurity  
@EnableMethodSecurity(prePostEnabled = true)  
public class SecurityConfig {
	...
}
```
