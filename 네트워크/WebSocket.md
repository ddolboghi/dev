# 특징
- WebSocket 프로토콜은 클라이언트와 서버가 전이중 채널에서 통신하는 방식이다.
- HTTP 폴링보다 오버헤드가 적어 실시간 애플리케이션에 적합하다.
- WebSocket은 클라이언트와 서버의 지속적인 연결 덕분에 상태를 저장할 수 있다. 이는 개발자가 WebSocket 연결을 사용하는 방식에 달려있다.
- WebSocket 메시지는 자주 변경되기 때문에 캐싱하기 불리하다.

# 동작 과정
![WebSocket protocol chart](../img/WebSocket-protocol-chart.jpg)

1. 접속 요청은 HTTP GET Method로 해서 