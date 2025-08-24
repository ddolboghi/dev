# Socket Programming with UDP
발신 프로세스가 데이터를 패킷에 담아 소켓을 통해 보내기 전에, 먼저 **패킷에 목적지 주소를 붙여야 한다.** 이 목적지 주소는 인터넷을 통해 패킷을 라우팅하여 수신 프로세스의 소켓으로 전달하는 데 사용된다. 패킷이 수신 소켓에 도착하면, 수신 프로세스는 소켓을 통해 패킷을 검색하고 그 내용을 확인한 후 적절한 조치를 취한다.

목적지 주소에는 **목적지 호스트의 IP 주소**와 **목적지 소켓의 포트 번호**가 포함된다. IP 주소는 인터넷의 라우터들이 패킷을 목적지 호스트로 라우팅하는 데 필요하며, 포트 번호는 하나의 호스트에서 실행될 수 있는 여러 네트워크 애플리케이션 프로세스 중 특정 소켓을 식별하는 데 필요하다. 
발신 측 IP 주소와 소켓 포트 번호로 구성된 **출발지 주소도 패킷에 포함**되는데, 이는 일반적으로 애플리케이션 코드가 아닌 운영 체제에 의해 자동으로 처리된다.

## Python Socket Program
### Client

```py
from socket import *
serverName = 'hostname'
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('Input lowercase sentence:')
clientSocket.sendto(message.encode(),(serverName, serverPort))
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage.decode())
clientSocket.close()
```
- `serverName = 'hostname'` 및 `serverPort = 12000`: 첫 번째 줄은 변수 `serverName`을 서버의 IP 주소(예: "128.138.32.126") 또는 호스트 이름(예: "cis.poly.edu")으로 설정한다. 호스트 이름을 사용하면 DNS 조회가 자동으로 수행된다. 두 번째 줄은 정수 변수 `serverPort`를 12000으로 설정한다.
- `clientSocket = socket(AF_INET, SOCK_DGRAM)`: 이 줄은 `clientSocket`이라는 클라이언트 소켓을 생성한다. 첫 번째 매개변수 `AF_INET`는 기반 네트워크가 IPv4를 사용함을 나타낸다. 두 번째 매개변수 `SOCK_DGRAM`은 이 소켓이 UDP 소켓임을 의미한다. 클라이언트 소켓의 포트 번호는 지정하지 않으며, 운영 체제가 자동으로 할당하도록 한다.
- `clientSocket.sendto(message.encode(), (serverName, serverPort))`: 이 줄은 메시지를 문자열에서 바이트 유형으로 변환한 후(`encode()`), 목적지 주소(`serverName`, `serverPort`)를 메시지에 붙여 `clientSocket`을 통해 패킷을 전송한다. 출발지 주소는 운영 체제에 의해 자동으로 패킷에 추가된다.
- `modifiedMessage, serverAddress = clientSocket.recvfrom(2048)`: 인터넷으로부터 클라이언트 소켓에 패킷이 도착하면, 패킷의 데이터는 `modifiedMessage` 변수에, 패킷의 출발지 주소(서버의 IP 주소와 포트 번호)는 `serverAddress` 변수에 저장된다. `recvfrom` 메서드는 버퍼 크기 2048을 입력으로 받는다.
- `clientSocket.close()`: 소켓을 닫고 프로세스를 종료한다.
### Server
```py
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))
print(”The server is ready to receive”)
while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    modifiedMessage = message.decode().upper()
    serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```
- `serverSocket.bind(('', serverPort))`: 포트 번호 12000을 서버의 소켓에 바인딩(할당)한다. 이로써 서버의 IP 주소와 포트 12000으로 전송되는 모든 패킷이 이 소켓으로 향하게 된다.
- `while True:`: 이 `while` 루프는 서버가 클라이언트로부터 패킷을 무한정 수신하고 처리할 수 있게 한다.
- `message, clientAddress = serverSocket.recvfrom(2048)`: 서버 소켓에 패킷이 도착하면, 패킷 데이터는 `message` 변수에, 출발지 주소(클라이언트의 IP 주소와 포트 번호)는 `clientAddress` 변수에 저장된다. 서버는 이 주소 정보를 응답을 보낼 반환 주소로 사용한다.    
- `serverSocket.sendto(modifiedMessage.encode(), clientAddress)`: 대문자로 변환된 메시지에 클라이언트 주소를 붙여 서버 소켓을 통해 패킷을 전송한다. 이 패킷은 인터넷을 통해 해당 클라이언트 주소로 전달된다.
# Socket Programming with TCP

UDP와 달리 TCP는 **연결 지향(connection-oriented)** 프로토콜이다. 이는 클라이언트와 서버가 데이터를 보내기 전에 먼저 핸드셰이크를 통해 TCP 연결을 설정해야 함을 의미한다. TCP 연결의 한쪽 끝은 클라이언트 소켓에, 다른 쪽 끝은 서버 소켓에 연결된다. 연결이 설정되면, 한쪽에서 다른 쪽으로 데이터를 보낼 때 단순히 데이터를 소켓을 통해 TCP 연결로 보내면 된다. 이는 UDP처럼 패킷에 목적지 주소를 붙여야 하는 것과 대조된다.

![[TCPServer_process_has_tow_sockets.png | 500]]
1. 서버는 클라이언트의 초기 접속을 받을 준비가 되어 있어야 한다. 이를 위해 서버 프로그램은 **welcoming socket**이라는 특별한 소켓을 가지고 있어야 하며, 이 소켓은 임의의 호스트에서 실행되는 클라이언트 프로세스로부터의 초기 접속을 기다린다.
2. 서버 프로세스가 실행 중이면, 클라이언트 프로세스는 서버에 TCP 연결을 시작할 수 있다. 클라이언트는 서버의 웰컴 소켓 주소(서버 호스트의 IP 주소와 소켓의 포트 번호)를 지정하여 TCP 소켓을 생성한다. 소켓 생성 후, 클라이언트는 3-way handshake를 시작하여 서버와 TCP 연결을 설정한다. 이 핸드셰이크 과정은 전송 계층 내에서 일어나며 클라이언트와 서버 프로그램에는 보이지 않는다.
3. 3-way handshake 동안 클라이언트 프로세스가 서버 프로세스의 웰컴 소켓에 "노크"를 하면, 서버는 해당 특정 클라이언트 전용의 새로운 소켓, 즉 **connection socket**을 생성한다.

애플리케이션 관점에서 클라이언트의 소켓과 서버의 연결 소켓은 파이프로 직접 연결된 것과 같다(Figure 2.29 참조). 클라이언트 프로세스는 자신의 소켓으로 임의의 바이트를 보낼 수 있으며, TCP는 서버 프로세스가 이 바이트들을 보낸 순서대로 연결 소켓을 통해 수신하도록 보장한다. 이처럼 TCP는 클라이언트와 서버 프로세스 간에 신뢰성 있는 서비스를 제공한다.