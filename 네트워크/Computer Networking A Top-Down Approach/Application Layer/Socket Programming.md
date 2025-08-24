# Socket Programming with UDP
발신 프로세스가 데이터를 패킷에 담아 소켓을 통해 보내기 전에, 먼저 **패킷에 목적지 주소를 붙여야 한다.** 이 목적지 주소는 인터넷을 통해 패킷을 라우팅하여 수신 프로세스의 소켓으로 전달하는 데 사용된다. 패킷이 수신 소켓에 도착하면, 수신 프로세스는 소켓을 통해 패킷을 검색하고 그 내용을 확인한 후 적절한 조치를 취한다.

목적지 주소에는 **목적지 호스트의 IP 주소**와 **목적지 소켓의 포트 번호**가 포함된다. IP 주소는 인터넷의 라우터들이 패킷을 목적지 호스트로 라우팅하는 데 필요하며, 포트 번호는 하나의 호스트에서 실행될 수 있는 여러 네트워크 애플리케이션 프로세스 중 특정 소켓을 식별하는 데 필요하다. 
발신 측 IP 주소와 소켓 포트 번호로 구성된 **출발지 주소도 패킷에 포함**되는데, 이는 일반적으로 애플리케이션 코드가 아닌 운영 체제에 의해 자동으로 처리된다.

## Python Socket Program

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
