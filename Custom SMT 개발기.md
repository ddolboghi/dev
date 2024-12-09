---
Created: 2024-12-09
---
# 참고
- [Create Custom Kafka Connect Single Message Transforms for Confluent Platform](https://docs.confluent.io/platform/current/connect/transforms/custom.html)
- [Replacing Confluent’s Kafka Connect Filter SMT with a Custom SMT in Java: A Step-by-Step Guide](https://medium.com/@maheshbhatm/replacing-confluents-kafka-connect-filter-smt-with-a-custom-smt-in-java-a-step-by-step-guide-e1882a93f132)
- [How to Use Single Message Transforms in Kafka Connect](https://www.confluent.io/blog/kafka-connect-single-message-transformation-tutorial-with-examples/?session_ref=https://www.google.com/&_ga=2.238432201.896469771.1733720144-1465531381.1731048925&_gac=1.150693572.1732606365.CjwKCAiA3ZC6BhBaEiwAeqfvygiZ0aZ1sqUgRIlk-bJnwRg-psgtuGxVoD9KophieCYBwugSOM6bMBoCTvYQAvD_BwE&_gl=1*1ukfh6g*_gcl_aw*R0NMLjE3MzI2MDYzNjUuQ2p3S0NBaUEzWkM2QmhCYUVpd0FlcWZ2eWdpWjBhWjFzcVVnUklsay1iSm53UmctcHNndHVHeFZvRDlLb3BoaWVDWUJ3dWdTT002Yk1Cb0NUdllRQXZEX0J3RQ..*_gcl_au*Njk0MDcyNjEzLjE3MzEwNDg5MjQ.*_ga*MTQ2NTUzMTM4MS4xNzMxMDQ4OTI1*_ga_D2D3EGKSGD*MTczMzcyMDE0NC40LjEuMTczMzcyMDE1MC41NC4wLjA.)
- [kafka connect transform github](https://github.com/apache/kafka/blob/trunk/connect/transforms/src/main/java/org/apache/kafka/connect/transforms/TimestampConverter.java)
- [Interface Transformation docs](https://docs.confluent.io/platform/current/connect/javadocs/javadoc/org/apache/kafka/connect/transforms/Transformation.html)
# 요구사항
postgresql의 timestamp without time zone 형식의 데이터를 ISO8601 형식으로 변환해서 저장한다.

# 개발 환경
- kafka 3.6.x 가정
- gradle build
- jdk 17

# 개발
메시지의 스키마를 보면 postgresql의 timestamp without time zone을`io.debezium.time.MicroTimestamp`으로 받아 `int64` 타입으로 변환하고 있었습니다.
```json
{"schema": 
	{"type":"struct","fields":
		[
		...{"type":"int64","optional":false,"name":"io.debezium.time.MicroTimestamp","version":1,"field":"created_at"},
		... 
		]
```

1970년 1월 1일 자정을 기준으로 ms로 나타낸 시각을 `yyyy-MM-dd'T'HH:mm:ss'Z'`형식으로 변환해야 합니다. Z는 UTC 시간이라는 뜻이며, 서버는 모두 UTC 시간을 사용하고 있었기 때문에 TIME ZONE을 뺀 UTC 시간으로 변환합니다.

## Single Message Transforms 작동 원리
- transformation들은 JAR로 컴파일되며 Connect worker의 속성 파일에 지정된 plugin.path를 통해 Kafka Connect에서 사용할 수 있습니다. 일단 설치되면 커넥터 속성에서 transform을 구성할 수 있습니다.
- transformation을 구성하고 배포하면, 소스 커넥터가 업스트림 시스템으로부터 레코드를 받아 ConnectRecord로 변환합니다. 그리고 transformation의 `apply()`함수에 레코드를 전달하고, 레코드를 반환 받으려고 합니다.
- 싱크 커넥터에서는 이 과정이 반대로 진행됩니다. 소스 카프카 토픽의 각 메세지를 읽고 역직렬화한 다음, transformation의 `apply()`함수이 호출되고 이를 거친 레코드가 타겟 시스템으로 보내집니다.
- `configure()`함수로 파라미터를 받을 수 있습니다.