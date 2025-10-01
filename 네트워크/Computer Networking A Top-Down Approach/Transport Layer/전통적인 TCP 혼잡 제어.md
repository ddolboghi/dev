TCP는 발신지가 자신에게 트래픽을 보내는 속도를 제한하는 방식으로 혼잡을 제어한다.
# 전송 속도 제한 방식
- 발신지는 **혼잡 윈도우(congestion window, `cwnd`)** 라는 변수로 전송소도를 제한한다.
- 발신지에서 아직 ACK를 받지 않은 데이터의 양은 `cwnd`와 `rwnd`(수신 윈도우) 중 작은 값 이하여야 한다.
  $LastByteSent - LastByteAcked \leq min(cwnd, rwnd)$
  이 제약 조건은 발신지의 전송 속도를 간접적으로 `cwnd/RTT` bytes/sec로 제한한다.
- 발신지는 `cwnd`값을 조절하여 전송 속도를 제어한다.