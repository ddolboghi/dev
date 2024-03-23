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
  client_id=CKw2bkLjI-6Bs3wwgl7OBUgz&
  redirect_uri=http://localhost:3000/api/auth/callback/provider&
  response_type=code&
  scope=photo+offline_access&
  state=w7pneFNa8aF2i5f_
```
- `client_id`(**필수**): 클라이언트 애플리케이션이 OAuth2 제공자로부터 발급 받은 고유 id입니다.
- `redirect_uri`(**필수**): 제공자가 인가 요청에 대한 응답을 전달할 리다이렉션 엔드포인트입니다.
- `response_type`(**필수**): athorization code grant flow를 사용함을 명시하기 위해 `code`라고 작성합니다. 
- `scope`: 클라이언트가 요청하는 사용자 정보에 대한 접근 범위입니다.
- `state`
: 클라이언트가 제공자에게 `state`값을 전달하면 제공자는 이 값을 다시 응답에 포함해서 전달합니다. CSRF(c.f. [[#쿠키의 `SameSite` 속성과 CSRF]])공격을 차단할 수 있습니다. 

3. OAuth2 제공자는 사용자에게 클라이언트 서버가 사용자의 리소스에 접근하도록 허용할지 직접 묻습니다.  

4. 사용자가 OAuth2 서버에 인증(로그인)합니다.

5. 인증이 성공하면 OAuth2 서버는 (A)에서 제공한 웹브라우저의 리다이렉트 URL로 `code`와 `state`를 전달합니다.
```
http://localhost:3000/api/auth/callback/provider?
	state=w7pneFNa8aF2i5f_
    &code=Uyz9EU-QeRfW4Kt-nUnq4s7NxMuFjJLhT3DVHD6VyLn8Mc5Q
```
- `code`: access token과 교환하는데 사용할 athorization code입니다.
- (A)에서 인가 요청 시 `state`를 같이 보냈다면 응답에도 같은 `state`가 존재합니다.

> [!tip]
> - athorization code는 1회용이므로 동일한 코드로 새로운 access token을 요청하면실패합니다.
> - athorization code는 만료시간이 짧으므로 클라이언트 서버는 코드를 곧바로 사용해야 합니다.
> - OAuth 2.0은 athorization code의 만료시간을 10분으로 제한하도록 권장합니다.

- 인가 요청이 거부되면 access token은 전달되지 않습니다.
- athorization code grant flow에서 인가 요청 시 에러 응답은 URL 쿼리 파라미터로 전달되지만, implicit grant flow는 URL fregment(/...#...)로 전달합니다.
```
HTTP/1.1 302 Found 
Location: {redirect_uri}? 
	error={error_code}&
	error_description={error_description}&
	error_uri={error_uri}&
	state={state}
```
- `error`(**필수**): 에러 코드
	- `invalid_request`: 요청 데이터가 잘못됨
	- `unauthorized_client`: 클라이언트 애플리케이션이 요청을 전달할 권한이 없음
	- `access_denied`: 사용자가 거부함
	- `unsupported_response_type`: 잘못된 응답 유형이 사용됨.(`response_type`을 잘못 지정함)
	- `server_error`: 서버 내에서 에러 발생으로 인가 요청이 처리가 안됨
	- `temporarily_unavailable`: 인가 서버가 일시적 장애 상태
- `error_description`: 사람이 읽을 수 있는 에러 메시지
- `error_uri`: 에러에 대한 자세한 정보가 있는 링크
- `state`: 인가 요청 시 클라이언트에서 제공한 값

6. 클라이언트가 웹 브라우저의 URL을 파싱해 인가 응답 값을 받습니다.

7. 클라이언트는 전달받은 `code`로 OAuth2 서버에 `access token`을 요청합니다.
```
POST /token HTTP/1.1 
Host: authorization-server.com 
Authorization: Basic [encoded_client_credentials] // 클라이언트 인증 
Content-type: application/x-www-form-urlencoded

grant_type=authorization_code&
code={authorization_code}&
redirect_uri={redirect_uri}&
client_id={client_id}
```
- `grant_type`(**필수**):  access token으로 교환하고자 한다는 것을 나타내기 위해 `authorization_code`으로 셋팅해야 합니다.
- `code`: (C)에서 받은 인가 코드 값입니다.
- `redirect_uri`(**필수**): 인가 요청에 리다이렉션 엔드포인트가 포함되었다면 액세스 토큰 요청에도 리다이렉션 엔드포인트가 포함되어야 한다.
- 클라이언트는 제공자에게 `client_id`, `client_secret`으로 인증합니다.

> [!tip] OAuth 2.0을 지원하는 기업은 HTTP 요청의 Athorization 헤더로 클라이언를확인하지만, 지원하지 않는 기업은 요청 파라미터로 클라이언트를 확인하기도 합니다.
> OAuth 2.0 이전의 클라이언트 인증 방식
> ```
> 	POST /token HTTP/1.1 
> 	Host: [server.exmple.com](http://server.exmple.com/)
> 	Content-type: application/x-www-form-urlencoded 
> 	
> 	grant_type=authorization_code&
> 	code={authorization_code}& 
> 	redirect_uri={redirect_uri}& 
> 	client_id={client_id}& 
> 	client_secret={client_secret}
> ```
> HTTP에 Authorization 헤더가 없고, 클라이언트 시크릿 파라미터를 함께 전달합니 다.

8. 제공자가 클라이언트에게 access token을 전달합니다.
- `access_token`: (필수) 인가 요청의 성공으로 얻은 access_token
- `token_type`: 전달되는 토큰의 유형. 거의 bearer 토큰 유형.
- `expires_in`: (선택) 토큰의 유효기(초 단위)
- `refresh_token`: (선택) 액세스 토큰이 만료돼었을때 갱신을 위한 토큰
- `scope`: (조건부 필수) 인가된 범위와 요청된 범위가 같다면 생략될 수 있음. 다르다면 인가된 범위가 전달된다.