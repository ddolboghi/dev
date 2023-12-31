# 토큰 기반 인증
토큰 : 서버에서 클라이언트를 구분하기 위한 유일한 값
- 서버가 토큰을 생성해서 클라이언트에게 제공하면 클라이언트는 이 토큰을 갖고 있다가 여러 요청을 이 토큰과 함께 신청하고 서버는 토큰만 보고 유효한 사용자인지 검증
## 토큰 전달 및 인증 과정
[토큰 기반 인증 다이어그램](obsidian://open?vault=Obsidian%20Vault&file=springboot%2F%ED%86%A0%ED%81%B0%20%EA%B8%B0%EB%B0%98%20%EC%9D%B8%EC%A6%9D.canvas)
1. <클라이언트>로그인 요청 : 클라이언트가 아이디와 비밀번호를 서버에 전달하면서 인증 요청
2. <서버>토큰 생성 후 응답 : 서버는 아이디와 비밀번호를 확인해 유효한 사용자인지 검증하고 유효한 사용자면 토큰을 생성해서 응답
3. <클라이언트>토큰 저장 : 클라이언트가 서버에서 준 토큰 저장
4. <클라이언트>토큰 정보와 함께 요청 : 클라이언트는 이후 인증이 필요한 API를 사용할때 토큰을 함께 보냄
5. <서버>토큰 검증 : 클라이언트가 보낸 토큰이 유효한지 검증
6. <서버>응답 : 토큰이 유효하다면 클라이언트가 보낸 요청 처리
## 토큰 기반 인증의 특징
### 무상태성
- 클라이언트에서 인증 정보가 담긴 토큰을 생성하고 인증하므로, 서버에 토큰을 저장할 필요 없음
- 클라이언트가 사용자의 인증 상태를 유지하는 상태 관리를 하므로 서버는 클라이언트의 인증 정보를 저장하거나 유지할 필요 없어 완전한 무상태(stateless)로 효율적인 검증 가능
### 확장성
- 무상태성으로 인해 서버가 상태 관리를 신경쓰지 않아도되므로 서버 확장에 용이
- 하나의 토큰으로 여러 인증 요청을 보낼수 있고 소셜 로그인이나 다른 서비스에 권한 공유 가능
### 무결성
- 토큰 방식은 HMAC(hash-based message authentication)기법이므로 토큰 발급 이후에는 토큰 정보를 변경할 수 없음. 즉 토큰의 무결성이 보장됨

# JWT 구조
- JWT 인증을 하려면 HTTP요청 헤더 중 Authorization키값에 Bearer + JWT 토큰값을 넣어 보내야함
JWT 구조 : ==헤더(header) . 내용(payload) . 서명(signature)==
## 헤더
- 토큰의 타입과 해싱 알고리즘을 지정하는 정보를 담음
- typ : 토큰의 타입 지정, "JWT" 지정
- alg : 해싱 알고리즘 지정

```
//JWT토큰, HS256 해싱 알고리즘 사용
{"typ": "JWT", "alg": "HS256"}
```
## 내용
- 토큰과 관련된 정보 담음
- claim : 키-값 한 쌍으로 이뤄진 내용의 한 덩어리

|registered claim(등록된 클레임)| |
|---|---|
|iss|토큰 발급자(issuer)|
|sub|토큰 제목(subject)|
|aud|토큰 대상자(audience)|
|exp|토큰 만료시간(expiraton). NumericDate형식으로 항상 현재 시간 이후로 설정|
|nbf|토큰의 활성 날짜와 비슷한 개념. NumericData형식으로 날짜 지정하고 이 날짜가 지나기 전까지는 토큰이 처리되지 않음(not before)|
|iat|토큰이 발급된 시간(issued at)|
|jti|JWT의 고유 식별자로서 주로 일회용 토큰에 사용|
- public claim(공개 클레임) : 공개되어도 상관없는 클레임. 충돌하지 않는 이름이어야하며 보통 URI로 지음
- private claim(비공개 클레임) : 공개되면 안되는 클레임. 클라이언트와 서버간의 통신에 사용
```JWT
{
	"iss": "test@test.com", //registered claim
	"iat": 1622370878, //registered claim
	"exp": 1622372678, //registered claim
	"https://siwoli.com/jwt_claims/is_admin": true, //public claim
	"email": "test@test.com", //private claim
	"hello": "hi" //private claim
}
```
## 서명
- 해당 토큰이 조작되었거나 변경되지 않았음을 확인하는 용도
- 헤더의 인코딩 값과 내용의 인코딩 값을 합친 후에 주어진 비밀키를 사용해 해시값 생성
# 리프레시 토큰
- ==서버는 토큰과 함께 들어온 요청이 토큰을 탈취한 사람의 요청인지 확인할 수 없음==
- 리프레시 토큰은 액세스 토큰이 만료되었을때 새로운 엑세스 토큰을 발급하기 위해 사용
- 엑세스 토큰의 유효기간은 짧게 설정하고 리프레시 토큰의 유효기간을 길게 설정
[리프레시 토큰 요청&응답 다이어그램](obsidian://open?vault=Obsidian%20Vault&file=springboot%2F%EB%A6%AC%ED%94%84%EB%A0%88%EC%8B%9C%20%ED%86%A0%ED%81%B0%20%EC%9A%94%EC%B2%AD%26%EC%9D%91%EB%8B%B5.canvas)
1. <클라이언트>클라이언트가 서버에 인증 요청
2. <서버>서버는 유효성 검사 후 엑세스 토큰과 리프레시 토큰을 만들어 응답
3. <서버>서버에서 생성한 리프레시 토큰은 클라이언트와 DB에 모두 저장
4. <클라이언트>인증을 필요로하는 API 호출 시 클라이언트에 저장된 액세스 토큰과 함께 API 요청
5. <서버>서버는 전달받은 액세스 토큰의 유효성 검사 후 요청 처리
6. <클라이언트>만료된 엑세스 토큰과 함께 요청
7. <서버>토큰 만료 에러 응답
8. <클라이언트>클라이언트가 저장해둔 리프레시 토큰과 함께 새로운 엑세스 토큰 발급 요청 보냄
9. <서버>전달받은 리프레시 토큰이 유효한지 DB에 저장해둔 리프레시 토큰과 비교
10. <서버>유효한 리프레시 토큰이면 새로운 엑세스 토큰을 생성한 뒤 응답
11. <클라이언트>4번 반복
# JWT 서비스 구현하기
## 1. 의존성 추가
```gradle
dependencies {
	...
	implementation 'io.jsonwebtoken:jjwt:0.9.1' //자바 JWT 라이브러리  
	implementation 'javax.xml.bind:jaxb-api:2.3.1' //XML문서와 java객체 간 매핑 자동화
}
```
## 2. 토큰 제공자 추가하기
