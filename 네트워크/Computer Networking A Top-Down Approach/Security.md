# 방화벽(firewall)
- 방화벽은 하드웨어와 소프트웨어의 조합으로, 조직의 내부 네트워크를 인터넷 전체로부터 격리하여 일부 패킷은 통과시키고 다른 패킷은 차단한다.
## 방화벽의 3가지 목표
1. 들어오고 나가는 모든 트래픽은 방화벽을 통과한다.
	- 네트워크의 단일 지점에 방화벽을 위치시키면 보안 접근 정책을 관리하고 시행하기 쉽다.
2. 로컬 보안 정책에 의해서 정의됨으로써 인가된 트래픽만 방화벽을 통과한다.
3. 방화벽 자체는 침투에 영향을 받지 않는다. 
	- 방화벽 자체는 네트워크에 연결된 장치다.
- 현재 방화벽은 라우터 안에 구현되며, SDN들을 사용해서 원격 컨트롤이 가능하다.
- 방화벽은 traditional packet filters, statefull packet filters, application gateway의 3가지 종류가 있다.
## Traditional packet filters
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