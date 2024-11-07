# Authorization Code Grant flow
- OAuth2의 표준 인가 방식입니다.
![[authorization-code-grant-flow.png]]
- Resource Owner: Client가 요청하는 리소스의 소유자로, 사용자입니다. 사용자의 리소스는 아이디, 비밀번호 등의 개인정보입니다.
- Client: 리소스를 요청하며, 디바이스나 WAS 등입니다.
- User-Agent: 웹 브라우저로, Authorization Code Grant Flow는 http redirection 기반으로 인가를 진행하기 때문에 Resource Owner의 User-Agent와 상호작용합니다.
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
> [!tip] 응답은 제공자에 따라 json, XML, Key-Value 등 다른 포맷으로 보낼 수 있어 제공자의 문서에응답 포맷을 확인해야 합니다.
- 응답 값
	- `access_token`(**필수**):  인가 요청의 성공으로 얻은 access token
	- `token_type`: 전달되는 토큰의 유형(거의 bearer 토큰 유형)
	- `expires_in`: 토큰의 유효기간(초 단위, 숫자)
	- `refresh_token`: 액세스 토큰이 만료되었을때 갱신을 위한 토큰
	- `scope`(**조건부 필수**):  요청하는 사용자 정보 범위로, 인가된 범위와 같다면 생략할 수 있지만 다르`다면 인가된 범위의 정보만 전달됩니다.

- 인가 요청이 거부되면 아래 파라미터를 포함해 HTTP 400을 반환합니다.
	- `error`: (필수) 에러 코드로서 요청이 실패한 이유를 나타낸다.
	- `invalid_request`: 요청 데이터가 잘못됨
	- `invalid_client`: 클라이언트 인증 실패
	- `invalid_grant`: 제공되 그랜트가 유효하지 않음
	- `unauthorized_client`: 클라이언트 애플리케이션이 요청을 전달할 권한이 없음
	- `unsupported_grant_type`: 지원하지 않는 그랜트 유형
	- `invalid_scope`: 전달된 권한이 유효하지 않음
	- `error_description`: (선택) 사람이 읽을 수 있는 형태의 에러 메시지
	- `error_uri`: (선택) 에러에 대한 자세한 정보위 웹 문서 링크
```
# 에러 응답 형태 
HTTP/1.1 400 Bad Request 
Content-Type: application/json;charset=UTF-8 
Cache-Control: no-store 
Pragma: no-cache 
{ 
	"error":"invalid_client" 
}
```
# OAuth 2.0 PKCE
- PKCE(Proof Key for Code Exchange)는 OAuth 2.0에 좀 더 강화된 보안을 제공해주는 Authorization Code Grant flow의 확장 버전입니다.
- Authorization Code Grant flow에 `code_challenge`와 `code_verifier`를 추가해 authorization code가 탈취 당했을때 access token을 발급하지 못하도록 막습니다. 

> [!info] `code_verifier` 생성 규칙
> - 48 ~ 128 글자수를 가진 랜덤 문자열로, A-Z, a-z, 0-9, -, ., \_, ~ 로만 구성됩니다.

> [!info] `code_challenge` 생성 규칙
> - 선택한 Hash 알고리즘으로 `code_verifier`를 해싱한 후 **base64 인코딩**을 한 값입니다.
> e.g. `Base64Encode(Sha256(Code Verifier))`
## PKCE flow
1. Authorization Code Grant flow의 2단계에서 `code_challenge`와 `code_challenge_method`(hash 함수 종류)를 URL 쿼리 파라미터에 추가합니다.
```
https://authorization-server.com/authorize?
  client_id=CKw2bkLjI-6Bs3wwgl7OBUgz&
  redirect_uri=http://localhost:3000/api/auth/callback/provider&
  response_type=code&
  scope=photo+offline_access&
  state=w7pneFNa8aF2i5f_&
  code_challenge=HVoKJYs8JruAxs7hKcG4oLpJXCP-z1jJQtXpQte6GyA&
  code_challenge_method=S256
  ```

2. 7단계에서 제공자에게 `code_verifier`를 전달합니다.
```
POST /token HTTP/1.1 
Host: authorization-server.com 
Authorization: Basic [encoded_client_credentials] // 클라이언트 인증 
Content-type: application/x-www-form-urlencoded

grant_type=authorization_code&
code={authorization_code}&
redirect_uri={redirect_uri}&
client_id={client_id}&
code_verifier={code_verifier}
```

3. 제공자가 7단계에서 받은 `chde_verifier`
를 2단계에서 받은 `code_challenge_method` 해싱함수로 해싱 후 base64인코딩해서 `code_challenge`와 비교합니다.

4. `code_verifier`를 변환한 값과 `code_challenge`값이 일치하면 access token을 발급합니다.