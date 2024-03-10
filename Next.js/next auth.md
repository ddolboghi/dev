- next auth의 가장 중요한 목적은 OAuth입니다.
- 로그인 후 리다이렉션되는 페이지를 만들어야합니다.
- [초기 설정](https://next-auth.js.org/configuration/initialization)
- 리다이렉트 주소: `/api/auth/callback/:provider`
# [공식문서](https://authjs.dev/?_gl=1*1owi9s4*_gcl_au*Mjk1OTM2NDEwLjE3MTAwNDY5NDA.)

# database adapter 설정하기
- Auth.js의 adapter는 사용자, 계정, 세션 등의 데이터를 저장하는 데 사용하려는 데이터베이스 또는 백엔드 시스템에 애플리케이션을 연결합니다. 
- 자체 데이터베이스에 사용자 정보를 유지해야 하거나 특정 플로우를 구현하려는 경우가 아니라면 어댑터는 선택 사항입니다. 예를 들어 이메일 제공업체는 인증 토큰을 저장할 수 있는 어댑터가 필요합니다.
- 공식 문서 > Getting started > Database Adapters > Official adapters > 연결
- Prisma 기준으로 설명합니다.
- 