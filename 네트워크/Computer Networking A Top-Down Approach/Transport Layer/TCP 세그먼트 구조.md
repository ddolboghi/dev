- TCP 세그먼트는 **헤더 필드**와 **데이터 필드**로 구성된다.
- 데이터 필드는 애플리케이션 데이터 덩어리를 포함하며, MSS에 의해 크기가 제한된다.
- TCP 헤더는 일반적으로 20byte다.

![[TCP_segment_structure.png | 600]]
- 32-bit sequence number 필드, 32-bit acknowledgment number 필드: TCP 송신자와 수신자신뢰성있는 데이터 전송 서비스를 구현하는데 사용된다.
- 16-bit 수신 윈도우 필드: 흐름 제어에 사용되며, 수신자가 수용할 수 있는 바이트 수를 나타낸다.
- 4-bit 헤더 길이 필드: TCP 헤더 길이를 