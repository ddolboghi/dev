# OAuth2 PKCE
- PKCE(Proof Key for Code Exchange)는 OAuth2에 좀 더 강화된 보안을 제공해주는 Authorization Code Grant flow의 확장 버전입니다.
# Authorization Code Grant flow
- OAuth2의 표준 인가 방식입니다.
![[authorization-code-grant-flow.png]]
- Resource Owner: Client가 요청하는 리소스의 소유자로, 사용자입니다. 사용자의 리소스는 아이디, 비밀번호 등의 개인정보입니다.
- Client: 리소스를 요청하며, 디바이스나 WAS 등입니다.
- User-Agent: 웹 브라우저로, Authorization Code Grant Flow는 http redirection 베스로 인가를 진행하기 때문에 Resource Owner의 User-Agent와 상호작용합니다.
- Authorization Server: OAuth2를 제공하는 서버로, 구글, 카카오, 네이버 등입니다.

1. 클라이언트가 사용자의 인가를 요청하기 위해 웹브라우저를 OAuth2 서버로 리다이렉션 시킵니다.
2. OAuth2 서버 인가 요청에 필요한 파라미터를 URL에 포함하여 전달합니다. 전달된 파라미터는 `application/x-www-form-urlencoded` 포맷으로 인코딩됩니다.
```
https://authorization-server.com/authorize?
  response_type=code
  &client_id=CKw2bkLjI-6Bs3wwgl7OBUgz
  &redirect_uri=http://localhost:3000/api/auth/callback/provider
  &scope=photo+offline_access
  &state=w7pneFNa8aF2i5f_
```
- `client_id`: 클라이언트 애플리케이션이 OAuth2 제공자로부터 발급 받은 고유 id입니다.
- `redirect_uri`: 제공자가 인가 요청에 대한 응답을 전달할 리다이렉션 엔드포인입니다.
- `scope`: 클라이언트가 요청하는 사용자 정보에 대한 접근 범위입니다.
- `state`
: 클라이언트가 제공자에게 전달하면 제공자는 이 값을 다시 응답에 포함해서 전달합니다. CSRF(c.f. [[#쿠키의 `SameSite` 속성과 CSRF]])공격을 차단할 수 있습니다. 

3. OAuth2 제공자는 사용자에게 클라이언트 서버가 사용자의 리소스에 접근하도록 허용할지 직접 묻습니다.  
4. 사용자가 OAuth2 서버에 인증(로그인)합니다.

(C) 
- 인증이 성공하면 OAuth2 서버는 (A)에서 제공한 웹브라우저의 리다이렉트 URL로 `Authorization Code`와 `state`를 전달합니다.
```
http://localhost:3000/api/auth/callback/provider?
	state=w7pneFNa8aF2i5f_
    &code=Uyz9EU-QeRfW4Kt-nUnq4s7NxMuFjJLhT3DVHD6VyLn8Mc5Q
```

(D, E) 
- 클라이언트는 전달받은 `Authorization Code`로 OAuth2 서버에 `access token`을 요청합니다.
```
POST https://authorization-server.com/token

grant_type=authorization_code
&client_id=CKw2bkLjI-6Bs3wwgl7OBUgz
&client_secret=F3n7fXMtVwGJ5lXqTmwUHoNUp6O0qN1YYjkRkrQ7ZD6Kbnvt
&redirect_uri=https://www.oauth.com/playground/authorization-code-with-pkce.html
&code=Uyz9EU-QeRfW4Kt-nUnq4s7NxMuFjJLhT3DVHD6VyLn8Mc5Q
```