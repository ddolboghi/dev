- TCP는 손실된 세그먼트를 복구하기 위해 타임아웃/재전송 메커니즘을 사용한다.
- 타임아웃은 Round-Trip Time(RTT)보다 길어야 불필요한 재전송을 피할 수 있다.
# RTT 추정 방식
## **SampleRTT**
- SampleRTT는 한 세그먼트가 전송된 시점부터 해당 세그먼트에 대한 ACK를 받을 때까지의 시간이다.
- 모든 세그먼트에 대해 SampleRTT를 측정하지 않고, 전송되었으나 아직 ACK를 받지 못한 세그먼트 중 하나에 대해서만 측정한다. 이로 인해 대략 RTT마다 한 번씩 새로운 SampleRTT 값이 산출된다.
## **EstimatedRTT**
- EstimatedRTT는 SampleRTT 값들의 평균이다.
- SampleRTT 값은 네트워크 혼잡 등으로 인해 변동이 심할 수 있으므로 평균값을 사용한다.
- $EstimatedRTT = (1 - \alpha) \cdot EstimatedRTT + \alpha \cdot SampleRTT$
- 위 공식은 지수 가중 이동 평균 방식으로, 최근 샘플에 더 큰 가중치를 둔다. ㄱ
