# 특징
- Server-sent events는 클라이언트가 HTTP 연결을 통해 서버로부터 데이터를 수신할 수 있도록 하는 서버 푸시 기술이다.
- 단방향 통신으로, 서버에서 클라이언트로만 데이터를 전송할 수 있다.
- HTTP 기반이므로 새로운 프로토콜을 구현할 필요 없고 WebSocket보다 가볍고 구현이 간단하다.
- 네트워크 오류로 연결이 끊기면 클라이언트가 자동으로 재연결을 시도하는 표준 메커니즘이 내장되있다.
- 서버는 특정 이벤트 유형을 지정하여 데이터를 전송할 수 있다.
- HTTP/1.1에서 최대 6개의 브라우저
- 클라이언트는 JavaScript의 EventSource API로 SSE를 구현할 수 있다.
- `EventSource.addEventListener('event 이름', callback)`으로 서버에서 정의한 이벤트를 받아 처리할 수 있다.
# 동작 과정
```mermaid
sequenceDiagram
    participant Client as 클라이언트
    participant Server as 서버

    Client->>Server: GET /events HTTP/1.1<br>Accept: text/event-stream
    Note right of Client: SSE 연결 요청

    Server-->>Client: HTTP/1.1 200 OK<br>Content-Type: text/event-stream<br>Connection: keep-alive
    Note left of Server: 연결 수락 및 유지

    loop 데이터 전송
        Server-->>Client: data: 첫 번째 메시지
        Note left of Server: 데이터 전송
        Client->>Client: onmessage 이벤트 처리
        Note right of Client: 메시지 수신 및 처리

        Server-->>Client: event: userupdate<br>data: {"user": "Alice"}
        Note left of Server: 특정 이벤트와 데이터 전송
        Client->>Client: userupdate 이벤트 처리
        Note right of Client: 사용자 업데이트 이벤트 수신 및 처리
    end

    Client->>Server: EventSource.close()
    Note right of Client: 클라이언트가 연결 종료
    Server-->>Client: (연결 종료)
    Note left of Server: 서버 측 연결 종료
```
# 예시 코드
> [!NOTE]
    > SSE는 단순한 텍스트 포로토콜을 사용하며, 브라우저에 의해 각 필드는 `\n`으로 구분되고 메시지는 `\n\n`로 구분된다.
    > `event`: 이벤트 이름. 생략 시 "message"라는 기본 이벤트 이름이 사용된다.
    > `data`: 이벤트 데이터. 여러 줄일 경우 자동으로 합쳐짐
    > `id`: Last-Event-ID
    > `retry: <ms>` 자동 재연결 간격

FastAPI 서버
```python
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
import time

app = FastAPI()

def event_stream():
    count = 0
    while True:
        time.sleep(2)
        count += 1
        # 커스텀 이벤트 이름: my_custom_event
        yield f"event: my_custom_event\ndata: {{\"message\": \"Hello {count}\"}}\n\n"

@app.get("/sse")
async def sse_endpoint(request: Request):
    return StreamingResponse(event_stream(), media_type="text/event-stream")
```

브라우저는 내부적으로 JS 이벤트 객체를 생성한다.
```js
new MessageEvent("my_custom_event", {
  data: '{"foo": "bar"}'
});
```

React
```jsx
import React, { useEffect } from 'react';

export default function App() {
  useEffect(() => {
    const eventSource = new EventSource("http://localhost:8000/sse");

	// 이벤트 리스너에 이벤트 전달됨
    eventSource.addEventListener("my_custom_event", (event) => {
      const data = JSON.parse(event.data);
      console.log("Received custom event:", data);
    });

    eventSource.onerror = (err) => {
      console.error("SSE connection error:", err);
      eventSource.close();
    };

    return () => {
      eventSource.close(); // clean up
    };
  }, []);

  return (
    <div>
      <h1>SSE Custom Event Test</h1>
      <p>Check the console for incoming events.</p>
    </div>
  );
}
```