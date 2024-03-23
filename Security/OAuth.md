# OAuth2 PKCE
- PKCE(Proof Key for Code Exchange)는 OAuth2에 좀 더 강화된 보안을 제공해주는 Authorization Code Grant flow의 확장 버전입니다.
![[authorization-code-grant-flow.png]]
- Resource Owner: Client가 요청하는 리소스의 소유자로, 사용자입니다. 사용자의 리소스는 아이디, 비밀번호 등의 개인정보입니다.
- Client: 리소스를 요청하며, 디바이스나 WAS 등입니다.
- User-Agent: 웹 브라우저로, Authorization Code Grant Flow는 http redirection 베스로 인가를 진행하기 때문에 Resource Owner의 User-Agent와 상호작용합니다.
- Authorization Server: OAuth2를 제공하는 서버로, 구글, 카카오, 네이버 등입니다.

(A) 클라이언트가 사용자의 인가를 요청하기 위해 웹브라우저를 OAuth2 서버로 리다이렉션 시킵니다.
