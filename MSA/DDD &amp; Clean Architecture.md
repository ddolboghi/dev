# Domain Driven Design
## 도메인이란?
- 개발하고자 하는 서비스의 기능 그 자체
- 소프트웨어 엔티티와 도메인은 따로 생각
## 주요 개념
bounded context: 기능들을 하나의 도메인으로 묶어 각 도메인을 철저하게 분리
context map: bounded context간의 관계
aggregate: 데이터 변경 단위, 라이프 사이클이 같은 도메인들을 한데 모아 놓은 집합
- aggregate는 여러 개의 테이블과 연관있을 수 있음
## 대표적인 아키텍처 3가지
- Layered: 비즈니스 로직이 거대해지면 어플리케이션 레이어가 오염됨
- Clean
- Hexagonal: 클린 아키텍처와 유사하지만 `port`라는 구현 개념 명시
# Hexagonal Architecture
- **비즈니스 로직이 표현 로직이나 데이터 접근 로직에 의존하지 않도록 하는게 핵심**
- 레이어트 아키텍처는 흐름 중심이고, 클린 아키텍처는 도메인 중심
- 도메인을 소프트웨어 엔티티나 DB에 종속되지 않도록 함
- 포트는 들어오고 나가는 aggregate로 구성 -> 들어오거나 나가는 데이터에 사용하는 DTO 세분화
- 어댑터는 포트가 바깥으로 연결되는 통로
- 데이터를 생성하는 쪽에 가까우면 도메인이라고 볼 수 있음
- hexagonal architecture는 DDD를 잘 실현할 수 있음
![[hexagonal-ex1.png]]
1. 작품에 대한 판매 시작 요청이 controller를 통해 service port로 들어옴
2. 어플리케이션 및 도메인 내부에서 해당 요청 처리
3. 처리 완료되면 repository에서 persistance adapter(DAO)로 데이터 변경 처리
4. notification 시스템쪽으로 알림을 위한 메시지 발행
