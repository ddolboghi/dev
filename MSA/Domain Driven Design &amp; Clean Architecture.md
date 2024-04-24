# DDD
## 도메인이란?
- 개발하고자 하는 서비스의 기능 그 자체
- 소프트웨어 엔티티와 도메인은 따로 생각
## 주요 개념
bounded context: 기능들을 하나의 도메인으로 묶어 각 도메인을 철저하게 분리
context map: bounded context간의 관계
aggregate: 데이터 변경 단위, 라이프 사이클이 같은 도메인들을 한데 모아 놓은 집합

- aggregate는 이벤트스토밍을 여러번 수정하면서 도출
## 대표적인 아키텍처 3가지
- Layered
- Clean
- Hexagonal
# Clean Architecture
- 레이어트 아키텍처는 흐름 중심이고, 클린 아키텍처는 도메인 중심
- 도메인을 소프트웨어 엔티티나 DB에 종속되지 않도록 함
- 포트는 들어오고 나가는 aggregate로 구성 -> 들어오거나 나가는 데이터에 사용하는 DTO 세분화
- 어댑터는 포트가 바깥으로 연결되는 통로
- 데이터를 생성하는 쪽에 가까우면 도메인이라고 볼 수 있음
- hexagonal architecture는 DDD를 잘 실현할 수 있음