# 특징
- WebSocket 프로토콜은 클라이언트와 서버가 전이중 채널에서 통신하는 방식이다.
- HTTP 폴링보다 오버헤드가 적어 실시간 애플리케이션에 적합하다.
- WebSocket은 클라이언트와 서버의 지속적인 연결 덕분에 상태를 저장할 수 있다. 이는 개발자가 WebSocket 연결을 사용하는 방식에 달려있다.
- WebSocket 메시지는 자주 변경되기 때문에 캐싱하기 불리하다.

# 동작 과정
![WebSocket protocol chart](../img/WebSocket-protocol-chart.jpg)
기본적으로 HTTP 기반으로 handshake를 시작한 후, TCP 기반의 지속 연결을 유지하여 실시간 통신이 가능하도록 한다.
## 1. 
클라이언트가 HTTP GET Method로 해서 websocket 프로토콜로 변경된다.
다음은 websocket 프로토콜로 변경되기 위한 HTTP 헤더다.
```
GET /chat HTTP/1.1
Host: localhost:8080 
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== 
Sec-WebSocket-Protocol: chat, superchat 
Sec-WebSocket-Version: 13 
Origin: http://localhost:9000
```
### GET /chat HTTP/1.1
websocket 통신 요청을 위해서는 HTTP 1.1 이상, GET Method를 사용해야 한다.
### Upgrade
프로토콜을 전환하기 위해 사용하는 헤더다.
websocket 접속 요청시 'websocket'이라는 값을 가