# DDD
## 도메인이란?
- 개발하고자 하는 서비스의 기능 그 자체
- 소프트웨어 엔티티와 도메인은 따로 생각
## 주요 개념
bounded context: 기능들을 하나의 서비스로 묶음
context map: 서비스간의 상호관계
aggregate: 엔티티가 되기 전에 가지는 데이터들

- aggregate는 이벤트스토밍을 여러번 수정하면서 도출
# Clean Architecture
- 레이어트 아키텍처는 흐름 중심이고, 클린 아키텍처는 도메인 중심