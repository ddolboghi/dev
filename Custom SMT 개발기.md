---
Created: 2024-12-09
---
# 참고
- https://docs.confluent.io/platform/current/connect/transforms/custom.html
- https://medium.com/@maheshbhatm/replacing-confluents-kafka-connect-filter-smt-with-a-custom-smt-in-java-a-step-by-step-guide-e1882a93f132

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

1970년 1월 1일 자정을 기준으로 ms로 나타낸 시각을 `yyyy-MM-dd'T'HH:mm:ss'Z'`형식으로 변환해야 한다.