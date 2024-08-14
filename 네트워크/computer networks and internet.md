# 네트워크의 서비스적 개념
- 인터넷에 연결된 엔드 시스템의 프로그램은 다른 엔드 시스템에서 실행 중인 프로그램에 데이터를 전달하도록 요청하는 방법을 지정하는 **socket interface**를 제공한다.
- internet socket interface는 인터넷이 대상 프로그램에 데이터를 전달할 수 있도록 전송 프로그램이 따라야 하는 일련의 규칙이다.
- internet socket interface는 우편 시스템과 유사하다.
	1. 편지를 쓴다 == 데이터를 넣는다.
	2. 수신인, 주소, 봉투를 닫는다. == 정해진 형식에 맞게 데이터를 포장한다.
	3. 우편함에 넣는다. == 통신 체계에 맞게 데이터를 보낸다.
- 인터넷에서 두 개 이상의 원격 개체가 통신하는 모든 활동은 프로토콜이 적용된다.
## network edge
- **엔드 시스템은 host**라고 부른다. 왜냐하면 웹브라우저 같은 프로그램을 실행하는 주체이기 때문이다.
- 데이터센터의 host를 blade라고 한다. 이 host들은 rack에 쌓여있고 각각의 rack은 20~40개의 blade를 가진다. rack들은 데이터센터 네트워크 설계를 사용해 상호 연결된다.
- host는 client와 server로 나눠질 수 있다.
## access network
- access network는 엔드 시스템에서 다른 엔드 시스템으로 가는 경로에서 _엔드 시스템을 첫 번째 라우터(edge router)에 물리적으로 연결하는 네트워크_ 다.
- 가장 널리 사용되는 광대역 주거용 엑세스는 digital subscriber line(DSL)과 케이블이다.
- 집은 통신사로부터 DSL 인터넷 액세스를 얻는다.
- 이때 통신사는 ISP(Internet Service Provider)가 된다.
- 