- TCP는 손실된 세그먼트를 복구하기 위해 타임아웃/재전송 메커니즘을 사용한다.
- 타임아웃은 Round-Trip Time(RTT)보다 길어야 불필요한 재전송을 피할 수 있다.
# RTT 추정 방식
## **SampleRTT**
- SampleRTT는 한 세그먼트가 전송된 시점부터 해당 세그먼트에 대한 ACK를 받을 때까지의 시간이다.
- 모든 세그먼트에 대해 SampleRTT를 측정하지 않고, 전송되었으나 아직 ACK를 받지 못한 세그먼트 중 하나에 대해서만 측정한다. 이로 인해 대략 RTT마다 한 번씩 새로운 SampleRTT 값이 산출된다.
## **EstimatedRTT**
- EstimatedRTT는 SampleRTT 값들의 평균이다.
- SampleRTT 값은 네트워크 혼잡 등으로 인해 변동이 심할 수 있으므로 평균값을 사용한다.
>$EstimatedRTT = (1 - \alpha) \cdot EstimatedRTT + \alpha \cdot SampleRTT$
- 위 공식은 지수 가중 이동 평균으로, **최근 샘플에 더 큰 가중치**를 둔다.
- 권장 $\alpha$ 값은 0.125다.
## **DevRTT**
- DevRTT는 SampleRTT가 EstimatedRTT로부터 얼마나 벗어나는지를 추정한 값이다. RTT의 변동성을 측정하기 위해 사용한다.
>$DevRTT = (1 - \beta) \cdot DevRTT + \beta \cdot | SampleRTT - EstimatedRTT |$
- 위 공식은 SampleRTT와 EstimatedRTT의 차이에 대한 지수 가중 이동 평균이다.
- 권장되는 $\beta$ 값은 0.25다.
- SampleRTT 값의 변동이 작으면 DevRTT는 작아지고, 변동이 크면 DevRTT는 커진다.
# 재전송 타임아웃 간격 설정 및 관리
TCP의 타임아웃 간격은 다음 공식으로 설정된다.
>$TimeoutInterval = EstimatedRTT + 4 \cdot DevRTT$

- 타임아웃 값은 EstimatedRTT 값보다 크거나 같아야 한다. 하지만 너무 크면 세그먼트가 손실됐을때 빠르게 재전송하지 못한다. 그래서 약간의 여분을 더해준다.
- SampleRTT 값의 변동이 클수록 DevRTT 값도 커지므로 이 여분 값도 증가한다.
- 권장 초기 TimeoutInterval 값은 1초다.
- 타임아웃이 발생하면 다음 세그먼트에 대한 조급한 타임아웃을 피하기 위해 TimeoutInterval 값은 두 배가 된다. 그러나 새로운 세그먼트가 수신되고 EstimatedRTT가 갱신되면 위 공식을 사용하여 다시 계산된다.
