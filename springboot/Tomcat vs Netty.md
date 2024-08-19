# Tomcat
- 명령형 프로그래밍 방식의 스프링 부트 웹 애플리케이션에 사용되는 기본 servlet엔진
- 3.0v부터 비동기식 요청 처리, 3.1v부터 non-blocking I/O 추가
# Netty
- 성능이 입증된 비동기식 non-blocking I/O 엔진
- spring webflux는 reactor를 기반으로 하며 netty를 기본 네트워크 엔진으로 사용
- 필요한 경우 호환 가능한 모든 servlet 3.1 엔진을 spring webflux 애플리케이션과 함께 사용할 수 있다.