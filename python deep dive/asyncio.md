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
