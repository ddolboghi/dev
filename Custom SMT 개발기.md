---
Created: 2024-12-09
---
# 참고
- https://docs.confluent.io/platform/current/connect/transforms/custom.html
- https://medium.com/@maheshbhatm/replacing-confluents-kafka-connect-filter-smt-with-a-custom-smt-in-java-a-step-by-step-guide-e1882a93f132
- [](https://www.confluent.io/blog/kafka-connect-single-message-transformation-tutorial-with-examples/?session_ref=https://www.google.com/&_ga=2.238432201.896469771.1733720144-1465531381.1731048925&_gac=1.150693572.1732606365.CjwKCAiA3ZC6BhBaEiwAeqfvygiZ0aZ1sqUgRIlk-bJnwRg-psgtuGxVoD9KophieCYBwugSOM6bMBoCTvYQAvD_BwE&_gl=1*1ukfh6g*_gcl_aw*R0NMLjE3MzI2MDYzNjUuQ2p3S0NBaUEzWkM2QmhCYUVpd0FlcWZ2eWdpWjBhWjFzcVVnUklsay1iSm53UmctcHNndHVHeFZvRDlLb3BoaWVDWUJ3dWdTT002Yk1Cb0NUdllRQXZEX0J3RQ..*_gcl_au*Njk0MDcyNjEzLjE3MzEwNDg5MjQ.*_ga*MTQ2NTUzMTM4MS4xNzMxMDQ4OTI1*_ga_D2D3EGKSGD*MTczMzcyMDE0NC40LjEuMTczMzcyMDE1MC41NC4wLjA.)
# 요구사항
postgresql의 timestamp without time zone 형식의 데이터를 ISO8601 형식으로 변환해서 저장한다.

# 개발 환경
- kafka 3.6.x 가정
- gradle build
- jdk 17

# 개발
메시지의 스키마를 보면 postgresql의 timestamp without time zone을`io.debezium.time.MicroTimestamp`으로 받아 `int64` 타입으로 변환하고 있었다.
```json
{"schema": 
	{"type":"struct","fields":
		[
		...{"type":"int64","optional":false,"name":"io.debezium.time.MicroTimestamp","version":1,"field":"created_at"},
		... 
		]
```

1970년 1월 1일 자정을 기준으로 ms로 나타낸 시각을 `yyyy-MM-dd'T'HH:mm:ss'Z'`형식으로 변환해야 한다. Z는 UTC 시간이라는 뜻이며, 서버는 모두 UTC 시간을 사용하고 있었기 때문에 TIME ZONE을 뺀 UTC 시간으로 변환한다.