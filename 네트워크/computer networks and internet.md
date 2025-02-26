# 네트워크의 서비스적 개념
- 인터넷에 연결된 엔드 시스템의 프로그램은 다른 엔드 시스템에서 실행 중인 프로그램에 데이터를 전달하도록 요청하는 방법을 지정하는 **socket interface**를 제공한다.
- internet socket interface는 인터넷이 대상 프로그램에 데이터를 전달할 수 있도록 전송 프로그램이 따라야 하는 일련의 규칙이다.
- internet socket interface는 우편 시스템과 유사하다.
	1. 편지를 쓴다 == 데이터를 넣는다.
	2. 수신인, 주소, 봉투를 닫는다. == 정해진 형식에 맞게 데이터를 포장한다.
	3. 우편함에 넣는다. == 통신 체계에 맞게 데이터를 보낸다.
- 인터넷에서 두 개 이상의 원격 개체가 통신하는 모든 활동은 프로토콜이 적용된다.
# network edge
- **엔드 시스템은 host**라고 부른다. 왜냐하면 웹브라우저 같은 프로그램을 실행하는 주체이기 때문이다.
- 데이터센터의 host를 blade라고 한다. 이 host들은 rack에 쌓여있고 각각의 rack은 20~40개의 blade를 가진다. rack들은 데이터센터 네트워크 설계를 사용해 상호 연결된다.
- host는 client와 server로 나눠질 수 있다.
# access network
- access network는 엔드 시스템에서 다른 엔드 시스템으로 가는 경로에서 _엔드 시스템을 첫 번째 라우터(edge router)에 물리적으로 연결하는 네트워크_ 다.
- 가장 널리 사용되는 광대역 주거용 엑세스는 digital subscriber line(DSL)과 케이블이다.
## DSL
- 집은 통신사로부터 DSL 인터넷 액세스를 얻는다.
- 이때 통신사는 ISP(Internet Service Provider)가 된다.
- 집의 DSL 모뎀은 전화선을 사용해 데이터를 교환한다. 
- 집에서 아날로그 신호를 고주파로 통신사의 지역 중앙 사무실로 보내면 DSLAM(Digital Subscriber Line Access Multiplexer)이 디지털 신호로 변환한다.
- DSLAM은 데이터와 전화 신호를 분리해서 데이터를 인터넷으로 보낸다.
- 수백~수천 개의 집이 하나의 DSLAM과 연결되어있다.
- DLS 표준 downstream, upstream 속도는 다양하게 정의되어있다.
- downstream, upstream속도가 달라 엑세스가 비대칭적이다.
- 최대 속도는 통신사 사무실 간의 거리, twisted-pair line의 게이지 및 전기 간섭에 따라 제한된다.
## cable internet access
- 케이블 인터넷 액세스는 케이블 텔레비전 회사의 기존 케이블 텔레비전 인프라를 활용한다.
- 컴퓨터와 ethernet 포트로 연결된 케이블 모뎀이 필요하다.
- 케이블 모뎀은 HFC 네트워크를 downstream과 upstream 채널로 나눈다.
- CMTS(Cable Modem Termination System)은 케이블 모뎀으로부터 받은 아날로그 신호를 디지털로 변환한다.
- CMTS는 DSL 처럼 upstream과 downstream 속도가 다르며, 보통 downstream 속도가 더빠르다.
- CMTS는 DSL 처럼 계약한 속도나 최고 속도로 사용하지 못할 수도 있다.
- **케이블 인터넷 엑세스는 broadcast 매체이므로 공유되어있다.** 
	- 여러 downstream 채널이 동시에 큰 데이터를 받으면 각 채널의 속도는 총 케이블 downstream 속도보다 매우 낮아진다.
- upstream 채널도 공유되므로 전송을 조정하고 충돌을 피하기 위해 분산 다중 액세스 프로토콜이 필요하다.
## Fiber To The Home(FTTP)
- 통신사 지역 중앙 사무소에서 각각의 집으로 광케이블을 직접 연결한다.
- 인터넷 접근 속도가 GB/S 단위다.
## ethernet과 wifi
- LAN(Local Area Network)은 엔드 시스템을 엣지 라우터에 연결하는 데 사용된다.
- ethernet은 가장 널리 사용되는 LAN 기술의 일종이다.
## 물리적 매체
- 비트(데이터)는 라우터를 통해 이동한다.
- 각 송신기-수신기 라우터 쌍에 대해 비트는 전자기파 또는 광학 펄스 형태로 물리적 매체를 통해 전송된다.
- 물리적 미디어는 유도 매체와 비유도 매체의 두 가지 범주로 나뉜다. 
	- 유도 매체에서는 광섬유 케이블, 트위스트 페어 구리선 또는 동축 케이블과 같은 고체 매체를 따라 파동이 유도된다. 
	- 비유도 매체를 사용하면 무선 LAN 또는 디지털 위성 채널과 같이 대기 및 우주 공간에서 전파가 전파된다.

| 유도 매체            |                                                                                                                          |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Twisted-Pair 구리선 | 가장 싸고 많이 사용된다.                                                                                                           |
| 동축(coaxial) 케이블  | - 구리선이 평행이 아닌 동심원 모양이다.<br>- 유도 공유 매체다.<br>- 여러 개의 엔드 시스템을 케이블에 직접 연결할 수 있다.<br>- 각 엔드 시스템은 다른 엔드 시스템에서 전송되는 모든 것을 수신한다. |
| 광섬유              | - 매우 빠른 비트 전송률<br>- 전자기 간섭에 영향을 받지 않는다.<br>- 최대 100킬로미터까지 신호 감쇠가 매우 낮다.<br>- 도청하기 매우 어렵다.<br>- 비싸다.                      |

| 비유도매체      |                                                                                         |
| ---------- | --------------------------------------------------------------------------------------- |
| 지상파 라디오 채널 | - 환경적 요소에 영향을 많이 받는다.<br>- 무선 LAN 기술은 근거리 무선 채널을 사용한다.<br>- 셀룰러 액세스 기술은 광역 무선 채널을 사용한다. |
| 위성 라디오 채널  | - 정지궤도 위성과 저궤도 위성으로 통신한다.<br>- 신호 지연이 크다.                                               |
# network core
- 네트워크 코어는 인터넷의 최종 시스템을 상호 연결하는 패킷 스위치와 링크의 메쉬다.
- 링크와 스위치 네트워크를 통해 데이터를 이동시킬때 circuit switching과 packet switching이라는 두 가지 기본 접근 방식이 있다.
## packet switching
- 각 패킷은 통신 링크와 패킷 스위치를 통해 이동한다.
- 패킷 스위치에는 **라우터**와 **링크 레이어 스위치**의 두 가지 주요 유형이 있다.
- 패킷은 각 통신 링크를 통해 링크의 전체 전송 속도와 동일한 속도로 전송된다.
- 패킷 스위치에는 여러 개의 링크가 연결되어 있다.
- source end system 또는 패킷 스위치가 전송 속도 R bit/sec의 링크를 통해 L bit 패킷을 전송하는 경우, 패킷 전송 시간은 L/R sec다.
	${패킷 크기}\div{패킷 스위치 전송 속도}={패킷 전송 시간}$
- packet switching은 링크를 통해 다른 패킷을 동시에 전송해야 하기 때문에 링크 중 하나가 혼잡한 경우 패킷은 버퍼에서 대기해야 하며 지연이 발생한다.
### Store-and-Forward Transmission
- 대부분의 패킷 스위치가 링크 입력에 사용한다.
- 패킷 스위치는 **패킷의 첫 번째 비트를 아웃바운드 링크로 전송하기 전에 전체 패킷을 수신**해야 합니다.
- 라우터는 일반적으로 들어오는 패킷을 나가는 링크로 전환하는 역할을 하기 때문에 많은 링크가 있다.
- 라우터는 패킷의 모든 비트를 **받아** **저장**하고 **처리**한 뒤에 아웃바운드 링크로 전송할 수 있다.
	![[store-and-forward packet switching]]
- Link는 end system과 packet switch, pecket switch와 pecket switch사이의 경로다.
- n개의 Link가 있으면 n-1개의 라우터가 있다.
- 전파 지연이 없고, source와 detination 사이에 n개의 링크가 있을때 패킷 1개의 전송 시간은 $n\frac{L}{R}$이다.
### 대기열 지연과 패킷 손실 (at store-and-forward transmission)
- 패킷 스위치는 연결된 링크마다 output buffer(output queue)를 가진다.
- output buffer에 해당 링크로 보내려는 패킷을 저장한다.
- 다른 패킷을 전송하는데 링크가 사용되고 있다면 패킷은 output buffer에서 대기한다.
- 패킷은 store-and-forward 지연에 output buffer 큐 지연도 겪는다.
- 큐 지연은 네트워크 혼잡도에 영향 받는다.
- **buffer 공간이 가득 차면** 해당 buffer에 도착한 패킷 또는 대기 중인 패킷 중 하나가 삭제되어 **패킷 손실**이 발생한다.
### Forwarding Tables and Routing Protocols
- 라우터가 패킷이 가야할 링크를 정해주는 것을 패킷 포워딩이라고 한다.
- 각 라우터에는 destination 주소(또는 일부)를 해당 라우터의 아웃바운드 링크에 매핑하는 forwarding table이 있다.
- 패킷이 라우터를 통해 dostination end system에 도달하는 과정은 지도 없이 사람들에게 물어가면서 목적지에 도착하려는 운전자와 유사하다.
	- 운전자 == 패킷
	- 래우터 == 묻는 사람
	- 사람들은 전체 경로의 일부 중 당장 가야할 가까운 경로를 알려준다.
1. source end system이 destination end system으로 패킷을 보낼때  destination의 IP 주소를 패킷 헤더에 포함시킨다.
2. 라우터는 패킷의 destination 주소를 검사한다.
3. 라우터는 forwarding table에서 dostination 주소에 인접한 라우터로 가는 아웃바운드 링크를 찾는다.
4. 라우터는 패킷을 아웃바운드 링크로 라우팅한다.
-  인터넷의 여러 특수 **routing protocol**로 forwarding table을 자동으로 설정한다.
- routing protocol은 라우터에서 목적지까지 가장 짧은 경로를 결정하고 이 경로를 forwarding table을 구성하는데 사용한다.
## circuit switching
- circuit-switched 네트워크에서는 엔드 시스템 간의 통신을 제공하기 위해 경로(버퍼, 링크 전송 속도)를 따라 필요한 **리소스**가 엔드 시스템 간의 통신 세션 기간 동안 **예약**되어 있다.
- packet-switched 네트워크에서는 이러한 리소스가 에약되어있지 않다.
- 엔드 시스템간 데이터가 전송되기 전에 먼저 연결해야하고, 이러한 연결을 회로(circuit)라고 한다.
- 네트워크는 회로를 설정할 때 연결 기간 동안 네트워크 링크에 일정한 전송 속도(각 링크의 전송 용량의 일부를 나타냄)를 예약한다.
- 전송 속도가 예약되있기 때문에 보장된 일정 속도로 데이터를 보낼 수 있다.
![[Pasted image 20240815001131.png]]
- 각 링크는 회로를 4개씩 가지고 있다.
- 각 회로는 4개의 동시 연결을 지원할 수 있다.
- 두 호스트가 연결되길 원할때 네트워크는 전용 end-to-end 연결을 설정한다.
- 각 링크에는 4개의 회로가 있으므로 링크 전송 용량의 4분의 1이 개별 end-to-end 연결에 사용된다.
	- 예를 들어 인접 스위치 간의 각 링크의 전송 속도가 1Mbps인 경우, 각 end-to-end circuit 스위치 연결의 전송 속도는 250kbps이다.
- circuit switching은 전용 회로가 유휴 시간 동안 유휴 상태이기 때문에 낭비가 발생한다.
- 회로를 설정하고 전송 용량을 예약하는 것이 복잡하고 경로를 따라 스위치의 작동을 조정하는 복잡한 시그널링 소프트웨어가 필요하다.
## Multiplexing in Circuit-Switched Networks
![[FDM and TDM.png]]
- 링크의 회로는 주파수 분할 다중화(FDM; frequency-division multiplexing) 또는 시분할 다중화(TDM; time-division multiplexing)로 구현된다.
- FDM을 사용하면 링크는 연결이 지속되는 동안 각 연결에 주파수 대역을 할당한다.
> [!tip]
> 주파수 대역의 폭을 대역폭(bandwidth)라고 한다.
- TDM은 고정된 기간의 프레임으로 시간을 나누고, 각 프레임은 고정된 수의 시간 슬롯으로 나뉜다.
- 네트워크가 링크를 통해 연결을 설정하면 네트워크는 매 프레임마다 하나의 시간 슬롯을 이 연결에 할당한다. 
- 이 슬롯은 해당 연결 전용으로 사용되며, 연결 데이터를 전송하는 데 사용할 수 있는 시간 슬롯이 매 프레임마다 하나씩 있다.
- 각 회로는 시간 슬롯 기간 동안 주기적으로 모든 대역폭을 확보한다.
- TDM에서 회로의 전송 속도 = 프레임 속도 X 슬롯의 비트 수
	- 프레임 전송 속도가 8000 frame/sec이고 각각의 슬롯이 8bit라면 전송 속도는 64kbps이다.
- ex) 호스트 A에서 호스트 B로 640,000 bits의 파일을 circuit switching으로 전송할때
	- 모든 링크가 24개의 슬롯을 가진 TDM을 사용하고 링크의 비트 전송률은 1.536 Mbps
	- 회로를 설정하는데 500 msec 소요
	- 각 회로의 전송 속도는 (1.536 Mbps) / 24 = 64 kbps
	- 파일 전송 속도는 (640,000 bits) / (64 kbps) + 500ms = 10.5sec
> [!NOTE]
> 전송 시간은 링크 개수와 상관없다.
## packet switching vs circuit switching

사용자가 10명인데 한 사용자가 갑자기 1,000 bits 패킷 1,000개를 생성하고 다른 사용자는 대기 상태로 패킷을 생성하지 않는다고 가정해 보자. 
circuit switching은 프레임당 10개의 슬롯이 있고 각 슬롯이 1,000비트로 구성된 TDM을 사용한다.
활성 사용자는 프레임당 하나의 시간 슬롯만 사용하여 데이터를 전송할 수 있고 각 프레임의 나머지 9개 시간 슬롯은 유휴 상태로 유지된다. 
활성 사용자의 100만 bits 데이터가 모두 전송되기까지는 10초가 걸린다. 
packet switching의 경우 활성 사용자의 패킷과 다중화해야 하는 패킷을 생성하는 다른 사용자가 없기 때문에 활성 사용자는 전체 링크 속도인 1Mbps로 패킷을 계속 보낼 수 있다. 
이 경우 활성 사용자의 모든 데이터는 1초 이내에 전송된다.

## network의 network
- access ISP들은 **네트워크의 네트워크**로 상호연결되있고, 이로써 엔드 시스템들간 패킷을 보낼 수 있다. 

# Application layer
## DNS
- 로컬 DNS 서버는 서버의 계층 구조에 강하게 속하지 않지만 DNS 아키텍처의 중심이다.
- 각 ISP는 로컬 DNS 서버를 가지며, 이는 default name server 라고도 불린다.
- 호스트가 ISP에 연결되면, ISP는 호스트에게 로컬 DNS 서버들의 한 개 이상의 IP 주소들을 제공한다.
- 호스트의 로컬 DNS 서버는 보통 호스트와 물리적으로 가까운 곳에 있다.
- 호스트가 DNS 쿼리를 하면 프록시 역할을 하는 로컬 DNS 서버로 쿼리가 전송되며, 로컬 DNS 서버는 DNS 서버 계층 구조로 쿼리를 전달한다.
158p 끝에서 4번째 줄부터
# Transport layer

# Network layer: 데이터 측면
## IP
### DHCP (Dynamic Host Configuration Protocol)
- 네트워크에 연결된 장치에 IP 주소와 기타 네트워크 설정 정보를 자동으로 할당하는 프로토콜
- DHCP를 이용하면 연결된 장치마다 수동으로 IP 주소를 할당하지 않아도 된다.
- DHCP는 기본 게이트웨이의 IP 주소도 제공한다.
- DHCP 서버는 엣지 라우터에 있다.
- DHCP 서버는 장치가 네트워크에 연결되면 IP 주소를 할당하고, 네트워크 연결이 끊기면 IP 주소를 회수한다.
### DHCP 프로토콜 과정(6.7.1)
chp 4의 4단계 중 마지막 두 단계만 실제로 필요하다.
1. 운영체제가 DHCP 요청 메시지를 UDP 세그먼트에 넣는다. 이때 destination 포트는 DHCP 서버 포트인 67이고, 소스 포트는 DHCP 클라이언트 포트인 68이다. UDP 세그먼트는 브로드캐스트 IP destination 주소와 0.0.0.0의 소스 IP 주소를 가지고 IP 데이터그램안에 위치한다.
2. IP 데이터그램은 이더넷프레임 내에 배치된다. 이더넷프레임은 destination MAC 주소를 가지고 스위치에 연결된 모든 장치들에 전파된다. 이때 이더넷프레임의 소스 MAC 주소는 내 컴퓨터의 MAC 주소다.
3. 2의 이더넷 프레임은 내 컴퓨터가 이더넷 스위치로 전송하는 첫번째 프레임이다. 스위치는 들어온 프레임을 라우터와 연결된 포트를 포함한 모든 아웃 포트로 전파한다.
4. 라우터는 자신의 MAC 주소를 가진 인터페이스에서 DHCP 요청이 포함된 이더넷프레임을 수신하고 이더넷프레임에서 IP 데이터그램을 추출한다. 데이터그램의 브로드캐스트 IP destination 주소는 이 IP 데이터그램이 이 노드에서 상위계층 프로토콜에 의해 처리되어야 함을 나타낸다. 따라서 데이터그램의 페이로드(UDP 세그먼트)는 UDP까지 demultiplexed 되고, DHCP 요청 메지가 UDP 세그먼트로부터 추출된다.
5. 라우터 안에서 돌아가고 있는 DHCP 서버는 CIDR 블록 내의 IP 주소를 할당한다. DHCP 서버는 DHCP ACK 메시지를 만드는데, 이 안에는 할당할 IP 주소, DNS 서버의 IP 주소, 기본 게이트웨이의 IP 주소, subnet block(네트워크 마스크)이 들어있다. DHCP 메시지는 UDP 세그먼트 안에 들어가고, UDP 세그먼트는 IP 데이터그램 안에 들어가며, IP 데이터그램은 이더넷프레임 안에 들어간다. 이더넷프레임 안에는 라우터가 속한 네트워크에 대한 라우터 인터페이스의 소스 MAC 주소와 내 컴퓨터의 destination MAC 주소가 들어있다.
6. DHCP ACK를 포함한 이더넷프레임은 라우터에 의해서 스위치로 보내진다. 스위치는 2에서 DHCP 요청이 포함된 이더넷프레임을 받은 적이 있기 때문에 프레임의 destination MAC 주소를 보고 내 컴퓨터로 연결되는 포트에만 전달한다.
7. 내 컴퓨터는 이더넷프레임을 수신하고나서 IP 데이터그램 -> UDP 세그먼트 -> DHCP ACK 메시지 순서로 추출한다. DHCP 클라이언트는 DHCP ACK 메시지 안에서 할당받은 IP 주소와 DNS 서버의 IP 주소를 기록한다. 그리고 기본 게이트웨이의 주소를 IP 포워딩 테이블에 설치한다. 그러면 이제부터 내 컴퓨터는 subnet 밖의 대상 주소를 갖는 모든 데이터그램을 기본 게이트웨이로 받게 된다. 이제 내 컴퓨터는 네트워크 컴포넌트로 초기화되고, 웹페이지 페치를 시작할 준비를 한다.
# 보안
## 방화벽(firewall)
- 방화벽은 하드웨어와 소프트웨어의 조합으로, 조직의 내부 네트워크를 인터넷 전체로부터 격리하여 일부 패킷은 통과시키고 다른 패킷은 차단한다.
### 방화벽의 3가지 목표
1. 들어오고 나가는 모든 트래픽은 방화벽을 통과한다.
	- 네트워크의 단일 지점에 방화벽을 위치시키면 보안 접근 정책을 관리하고 시행하기 쉽다.
2. 로컬 보안 정책에 의해서 정의됨으로써 인가된 트래픽만 방화벽을 통과한다.
3. 방화벽 자체는 침투에 영향을 받지 않는다. 
	- 방화벽 자체는 네트워크에 연결된 장치다.
- 현재 방화벽은 라우터 안에 구현되며, SDN들을 사용해서 원격 컨트롤이 가능하다.
- 방화벽은 traditional packet filters, statefull packet filters, application gateway의 3가지 종류가 있다.
### Traditional packet filters
- 라우터 안에서 패킷 필터링이 일어난다.
- 패킷 필터는 각 데이터그램을 개별적으로 검사한다.
- 관리자가 지정한 규칙에 따라 데이터그램을 통과시킬지 아니면 삭제할지 결정한다.
- 필터링 요소들:
	- 소스 IP 주소, 대상 IP 주소
	- IP 데이터그램 필드의 프로토콜 타입(TCP, UDP, ICMP, OSPF 등)
	- TCP, UDP의 소스 포트와 대상 포트
	- TCP 플래그 비트(SYN, ACK 등)
	- ICMP 메시지 타입
	- 네트워크를 들어오고 나가는 데이터그램의 다른 규칙들
	- 다른 라우터 인터페이스의 다른 규칙들
- 필터링 정책은 주소와 포트 번호의 조합에 기반한다.
700p 5번째 줄부터

### Statefull packet filters

### Application gateway


[Access Control List1](https://eveningdev.tistory.com/65)
[Access Control List2](https://jdcyber.tistory.com/17)
