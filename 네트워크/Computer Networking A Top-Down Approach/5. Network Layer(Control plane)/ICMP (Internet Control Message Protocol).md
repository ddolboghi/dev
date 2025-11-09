# ICMP 소개 및 위치
- ICMP는 호스트와 라우터가 서로 네트워크 계층 정보를 통신하기 위해 사용된다.
- 일반적으로 라우터는 ICMP 메시지를 생성하여 호스트에 오류 정보를 송신한다.
- ICMP는 IP 바로 위에 위치하며 ICMP 메시지는 IP 페이로드로 전달된다.
- 호스트가 ICMP를 상위 계층 프로토콜(프로토콜 번호 1)로 지정하는 IP 데이터그램을 수신하면, 데이터그램의 내용을 ICMP로 역다중화한다.
# ICMP 메시지 형식
- ICMP 메시지에는 타입(type)과 코드(code) 필드가 있다.
- 메시지에는 원본 IP 데이터그램의 헤더와 처음 8바이트가 있는데, 이는 송신자가 오류를 일으킨 데이터그램을 판별할 수 있게 한다.
### ICMP 메시지 타입

| ICMP 타입 | 코드  | 설명                                                                         |
| ------- | --- | -------------------------------------------------------------------------- |
| 0       | 0   | echo 응답(ping)                                                              |
| 3       | 0   | destination network unreachable(라우터가 목적지로 가는 경로를 찾지 못함)                    |
| 3       | 1   | destination host unreachable                                               |
| 3       | 2   | destination protocol unreachable                                           |
| 3       | 3   | destination port unreachable                                               |
| 3       | 6   | destination network unknown                                                |
| 3       | 7   | destination host unknown                                                   |
| 4       | 0   | source quench(혼잡 제어; 혼잡한 라우터가 이 메시지를 보내 호스트의 전송 속도를 줄이도록 강제하지만 거의 사용되지 않음) |
| 8       | 0   | echo request                                                               |
| 9       | 0   | router advertisement                                                       |
| 10      | 0   | router discovery                                                           |
| 11      | 0   | TTL expired                                                                |
| 12      | 0   | IP haeder bad                                                              |
# ping과 ICMP
- ping 프로그램은 목적지 호스트에 ICMP 타입 8, 코드 0 메시지를 보낸다.
- 목적지 호스트가 이 요청을 받으면,  타입 0, 코드 0의 ICMP 에코 응답을 보낸다.
- 대부분의 TCP/IP 구현은 운영체제에서 별도 프로세스가 아닌, ping 서버를 지원한다.
# Traceroute와 ICMP
- Traceroute 프로그램은 ICMP 메시지를 사용하여 구현된다.
- Traceroute는 출발지에서 목적지까지의 라우터 이름과 주소를 파악하기 위해 목적지로 일반 IP 데이터그램(사용되지 않는 포트번호를 가진 UDP 세그먼트)을 보낸다.
- 라우터는 TTL이 만료된 데이터그램을 폐기하고 출발지로 라우터의 이름과 IP 주소가 포함된 ICMP 메시지(타입 11, 코드0)를 보낸다. 그러면 출발지 호스트는 타이머를 통해 RTT를 얻고 ICMP 메시지로부터 라우터의 이름과 IP 주소를 획득한다.
- 출발지 호스트에서 보내는 UDP 세그먼트의 TTL은 보낼때마다 점점 증가하므로, 목적지 호스트로부터 ICMP 타입 3/코드 3 메시지를 받으면 더이상 패킷을 보내지 않는다.
