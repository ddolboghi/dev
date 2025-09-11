`asyncio`의 이벤트 루프, 태스크(Task), 퓨처(Future)와 같은 핵심 구성 요소들이 내부적으로 어떻게 상호작용하여 협력적 멀티태스킹(Cooperative Multitasking)을 구현하는지 설명한다.
나아가 `asyncio`의 이벤트 루프가 운영체제의 I/O 멀티플렉싱 기능을 어떻게 활용하는지, 그리고 `Task`와 `Future` 객체가 콜백 체인을 통해 어떻게 코루틴의 실행을 조율하는지를 `Lib/asyncio` 디렉터리의 소스 코드를 중심으로 상세히 해부한다.
# Event Loop
이벤트 루프는 `asyncio` 애플리케이션의 핵심이며, 모든 비동기 태스크와 콜백을 관리하고, 네트워크 I/O와 같은 저수준 작업을 처리한다.

| 클래스 이름              | 소스 파일                | 핵심 역할                                                             | 주요 메서드                                    |
| ------------------- | -------------------- | ----------------------------------------------------------------- | ----------------------------------------- |
| `AbstractEventLoop` | `base_events.py`     | 모든 이벤트 루프가 구현해야 할 인터페이스 명세 정의                                     | `run_forever`, `call_soon`, `create_task` |
| `SelectorEventLoop` | `selector_events.py` | 유닉스 계열 시스템의 기본 이벤트 루프. `selectors` 모듈을 사용하여 효율적인 I/O 멀티플렉싱을 수행한다. | `_run_once`                               |
| `Future`            | `futures.py`         | 비동기 연산의 최종 결과를 나타내는 저수준 어웨이터블. 콜백 기반 코드와 `await`를 연결하는 다리 역할을 한다. | `set_result`, `add_done_callback`         |
| `Task`              | `tasks.py`           | `Future`의 하위 클래스로, 코루틴을 감싸고 실행을 주도한다.                             | `__step`, `__wakeup`                      |
## 추상 이벤트 루프: base_events.py
`Lib/asyncio/base_events.py`에 정의된 `AbstractEventLoop` 추상 클래스는 이벤트 루프가 수행해야 할 핵심 기능들을 메서드로 정의한다.
- `run_forever()`: 이벤트 루프를 시작하여 `stop()`이 호출될 때까지 무한히 실행한다.
- `run_until_complete(future)`: 주어진 퓨처(또는 코루틴)가 완료될 때까지 이벤트 루프를 실행한다.
- `call_soon(callback, *args)`: 다음 이벤트 루프 반복(iteration/tick)에서 즉시 실행할 콜백을 스케줄링한다.  
- `call_later(delay, callback, *args)`: 지정된 `delay` 초 후에 실행될 콜백을 스케줄링한다.  
- `create_task(coro)`: 코루틴을 `Task` 객체로 감싸고 실행을 스케줄링한다.

이러한 추상 메서드들은 `asyncio`의 고수준 API가 특정 이벤트 루프 구현(예: 유닉스의 `SelectorEventLoop` 또는 윈도우의 `ProactorEventLoop`)에 종속되지 않도록 보장한다.
## 구체적인 구현: SelectorEventLoop
유닉스 계열 운영체제(리눅스, macOS 등)에서 `asyncio`의 기본 이벤트 루프는 `Lib/asyncio/selector_events.py`에 구현된 `SelectorEventLoop`이다. 이 클래스의 이름에서 알 수 있듯이, Python의 표준 라이브러리인 `selectors` 모듈을 사용한다.

`selectors` 모듈은 운영체제가 제공하는 가장 효율적인 I/O 멀티플렉싱 메커니즘에 대한 고수준 추상화를 제공한다. 예를 들어, 리눅스에서는 `epoll`, macOS나 BSD에서는 `kqueue`, 그 외 다른 유닉스 시스템에서는 전통적인 `select` 시스템 콜을 내부적으로 사용한다.

> [!tip]
    > I/O 멀티플렉싱이란, 여러 개의 I/O 채널(파일 디스크립터, 소켓 등)을 동시에 감시하다가, 그중 하나라도 읽거나 쓸 준비가 되면 즉시 알려주는 기능이다.

`SelectorEventLoop`는 초기화 시 `selectors.DefaultSelector()`를 호출하여 현재 시스템에서 가장 효율적인 셀렉터를 생성하고, 이를 `self._selector` 인스턴스 변수에 저장한다. 이후 모든 네트워크 I/O 작업은 이 셀렉터에 파일 디스크립터와 관련 콜백을 등록하는 방식으로 이루어진다.