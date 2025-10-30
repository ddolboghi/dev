# Autonomous System
- 규모 문제: 인터넷 네트워크 규모는 라우터에 라우팅 정보를 저장하고, 경로 계산에 막대한 오버헤드가 발생한다.
- 관리 자치성 문제: ISP는 네트워크 내부를 숨기면서 원하는 대로 네트워크를 운영하고 관리할 수 있어야 한다.
- 이 두 가지 문제는 라우터를 자율 시스템(Autonomous System, AS)으로 조직하여 해결될 수 있다.
- 자율 시스템은 **동일한 관리 제어하에 있는 라우터 그룹**으로 구성된다.
- AS는 전 세계적으로 고유한 자율 시스템 번호(Autonomous System Number, ASN)로 식별된다.
- 동일한 AS 내의 라우터들은 모두 동일한 라우팅 알고리즘을 실행하며 서로에 대한 정보를 가지고 있다.
- 자율 시스템 내에서 실행되는 라우팅 알고리즘을 **AS 내부 라우팅 프로토콜(intra-autonomous system routing protocol)** 이라고 한다.
# Open Shortest Path First (OSPF)
