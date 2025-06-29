---
chapter: "2"
---
프롬프트: 
다음 chapter들의 내용을 요약하지 말고 각 chapter 별로 그대로 설명해줘. 설명 중에 참고할 Fiqure가 있다면 'Fiqure 1.18'처럼 몇번 Fiqure인지 알려줘.:
chapter 2.1.1
chapter 2.1.2
chapter 2.1.3
chapter 2.1.4

## DNS
- 로컬 DNS 서버는 서버의 계층 구조에 강하게 속하지 않지만 DNS 아키텍처의 중심이다.
- 각 ISP는 로컬 DNS 서버를 가지며, 이는 default name server 라고도 불린다.
- 호스트가 ISP에 연결되면, ISP는 호스트에게 로컬 DNS 서버들의 한 개 이상의 IP 주소들을 제공한다.
- 호스트의 로컬 DNS 서버는 보통 호스트와 물리적으로 가까운 곳에 있다.
- 호스트가 DNS 쿼리를 하면 프록시 역할을 하는 로컬 DNS 서버로 쿼리가 전송되며, 로컬 DNS 서버는 DNS 서버 계층 구조로 쿼리를 전달한다.
158p 끝에서 4번째 줄부터