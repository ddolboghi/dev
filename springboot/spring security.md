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
[SecutiryFilterChain 다이어그램](obsidian://open?vault=Obsidian%20Vault&file=spring%20secutiry%20-%20SecurityFilterChain.canvas)
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
[폼 로그인 인증 절차 다이어그램](obsidian://open?vault=Obsidian%20Vault&file=spring%20secutiry%20-%20%ED%8F%BC%20%EB%A1%9C%EA%B7%B8%EC%9D%B8.canvas)
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


## 5. 회원가입 템플릿

## 6. 중복 회원가입 처리

