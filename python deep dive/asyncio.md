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

`BaseEventLoop 클래스는 AbstractEventLoop` 추상 클래스를 상속받고, 특정 이벤트 루프 구현(예: 유닉스의 `SelectorEventLoop` 또는 윈도우의 `ProactorEventLoop`)들은 `BaseEventLoop`를 상속받아, `asyncio`의 고수준 API가 이런 특정 이벤트 루프 구현에 종속되지 않도록 보장한다.
## 구체적인 구현: SelectorEventLoop
유닉스 계열 운영체제(리눅스, macOS 등)에서 `asyncio`의 기본 이벤트 루프는 `Lib/asyncio/selector_events.py`에 구현된 `SelectorEventLoop`이다. 이 클래스의 이름에서 알 수 있듯이, Python의 표준 라이브러리인 `selectors` 모듈을 사용한다.

`selectors` 모듈은 운영체제가 제공하는 가장 효율적인 I/O 멀티플렉싱 메커니즘에 대한 고수준 추상화를 제공한다. 예를 들어, 리눅스에서는 `epoll`, macOS나 BSD에서는 `kqueue`, 그 외 다른 유닉스 시스템에서는 전통적인 `select` 시스템 콜을 내부적으로 사용한다.

> [!tip]
    > I/O 멀티플렉싱이란, 여러 개의 I/O 채널(파일 디스크립터, 소켓 등)을 동시에 감시하다가, 그중 하나라도 읽거나 쓸 준비가 되면 즉시 알려주는 기능이다.

`SelectorEventLoop`는 초기화 시 `selectors.DefaultSelector()`를 호출하여 현재 시스템에서 가장 효율적인 셀렉터를 생성하고, 이를 `self._selector` 인스턴스 변수에 저장한다. 이후 모든 네트워크 I/O 작업은 이 셀렉터에 파일 디스크립터와 관련 콜백을 등록하는 방식으로 이루어진다.
## 메인 루프 로직: `_run_once()`와 `run_forever()`
`BaseEventLoop`의 `run_forever()` 메서드의 구현은 매우 단순하다. 본질적으로 `self._stopping` 플래그가 설정될 때까지 `self._run_once()`를 반복적으로 호출하는 무한 루프다.
```Python
# Lib/asyncio/base_events.py (BaseEventLoop)
def run_forever(self):
    self._run_forever_setup()
    try:
        while True:
            self._run_once()
            if self._stopping:
                break
    finally:
        self._run_forever_cleanup()
```

실질적인 모든 작업은 `_run_once()` 메서드 안에서 이루어진다. 이 메서드는 이벤트 루프의 한 '틱(tick)' 또는 '반복(iteration)'에 해당한다.

1. 타임아웃 계산
```Python
timeout = None
if self._ready or self._stopping:
    timeout = 0
elif self._scheduled:
    # Compute the desired timeout.
    timeout = self._scheduled[0]._when - self.time()
    if timeout > MAXIMUM_SELECT_TIMEOUT:
        timeout = MAXIMUM_SELECT_TIMEOUT
    elif timeout < 0:
        timeout = 0
```
먼저 스케줄링된 작업들 중 가장 가까운 미래에 실행될 작업의 시간을 확인하여 `select()` 호출의 `timeout` 값을 계산한다. 만약 즉시 실행해야 할 작업(`_ready` 큐)이 있거나 I/O 대기 작업이 없다면 `timeout`은 0이 된다.

2. I/O 이벤트 대기 (블로킹)
```Python
event_list = self._selector.select(timeout)
self._process_events(event_list)
```
`event_list = self._selector.select(timeout)`를 호출한다. 이것이 이벤트 루프 전체에서 유일하게 블로킹될 수 있는 지점이다. 이 호출은 운영체제 커널에 제어권을 넘긴다. 프로세스는 다음 두 가지 조건 중 하나가 충족될 때까지 휴면 상태(sleep)에 들어간다:
- 지정된 `timeout`이 만료될 때
- 셀렉터에 등록된 파일 디스크립터 중 하나에서 I/O 이벤트(예: 소켓에 데이터 도착)가 발생할 때
이 대기 시간 동안 Python 프로세스는 CPU를 전혀 소모하지 않는다.

3. 준비된 콜백 처리
```Python
end_time = self.time() + self._clock_resolution # 시스템이 정한 고정 기준점부터 경과된 시간 + 최소 단위(정밀도)
while self._scheduled:
    handle = self._scheduled[0]
    if handle._when >= end_time:
        break
    handle = heapq.heappop(self._scheduled)
    handle._scheduled = False
    self._ready.append(handle)
```
`select()`가 반환되면, `event_list`에는 I/O 준비가 완료된 파일 디스크립터와 해당 이벤트 정보가 담겨 있다. 이 리스트를 순회하며, 각 파일 디스크립터에 연결된 핸들을 `self._ready` 큐로 옮긴다.

4. 콜백 실행:
```Python
ntodo = len(self._ready)
for i in range(ntodo):
    handle = self._ready.popleft()
    if handle._cancelled:
        continue
    if self._debug:
        try:
            self._current_handle = handle
            t0 = self.time()
            handle._run()
            dt = self.time() - t0
            if dt >= self.slow_callback_duration:
                logger.warning('Executing %s took %.3f seconds',
                               _format_handle(handle), dt)
        finally:
            self._current_handle = None
    else:
        handle._run()
handle = None  # Needed to break cycles when an exception occurs.
```
`_ready` 큐에 있는 모든 핸들(I/O 이벤트로 인해 준비된 것, `call_soon`으로 예약된 것, 타이머 만료로 준비된 것 모두 포함)을 순회하며 하나씩 실행한다. 이 콜백들이 바로 코루틴의 실행을 한 단계 진행시키거나(`Task.__step`), 중단된 태스크를 깨우는(`Task.__wakeup`) 역할을 한다.

이 구조는 이벤트 루프가 코루틴이나 `await`의 존재를 직접 알지 못하는, 상대적으로 수동적인 I/O 멀티플렉서이자 콜백 실행기임을 보여준다. 루프의 역할은 운영체제로부터 I/O 이벤트나 타이머 이벤트를 기다렸다가, 그 이벤트에 등록된 콜백을 실행시켜주는 것뿐이다.

애플리케이션은 대부분의 시간을 OS 커널의 `select` 또는 `epoll` 호출 내부에서 CPU를 전혀 사용하지 않고 대기 상태로 보낸다. 실제 처리할 작업(I/O 준비 완료)이 있을 때만 깨어나 필요한 만큼만 작업을 수행하고 다시 대기 상태로 돌아간다. 따라서 `asyncio`는 I/O 바운드 작업을 효율적으로 수행할 수 있다. 이는 유휴 상태의 스레드조차도 컨텍스트 스위칭 오버헤드를 유발하는 스레딩 모델과 극명한 대조를 이룬다.

# 동시성 조율: `Future`& `Task`
이 두 객체는 `asyncio`에서 협력적 멀티태스킹을 구현하는 가장 핵심적인 메커니즘이다.
코루틴을 중단하고 재개하는 로직은 이벤트 루프 자체가 아니라, `Task` 객체에 내장되어 있다.
## `Future`
Lib/asyncio/futures.py에 정의된 `Future` 클래스는 `asyncio`의 가장 기본적인 어웨이터블 프리미티브이다.
`Future`는 아직 사용할 수 없는 미래의 결과값을 나타내는 'placeholder' 객체다. 이는 콜백 기반의 저수준 코드(예: 네트워크 프로토콜 구현)와 `async`/`await`를 사용하는 고수준 코드를 연결하는 다리 역할을 한다.
`Future` 객체는 내부적으로 간단한 상태 멤버 변수인 `_state`를 가지고 있으며, 주요 상태는 다음과 같다:
- `PENDING`: 아직 결과가 설정되지 않은 초기 상태.
- `CANCELLED`: `cancel()` 메서드에 의해 작업이 취소된 상태.
- `FINISHED`: 결과 또는 예외가 설정되어 작업이 완료된 상태.

`Future`의 동작을 이해하기 위해 핵심 메서드들은 다음과 같다:
- `set_result(result)`: 이 메서드는 `Future`의 상태를 `PENDING`에서 `FINISHED`로 바꾸고, 주어진 `result`를 내부적으로 저장한다.
  가장 중요한 동작은 `__schedule_callbacks()`를 호출하여 `Future`에 등록된 모든 '완료 콜백'을 순회하며 `loop.call_soon(callback, ...)`을 통해 이벤트 루프에서 실행되도록 스케줄링하는 것이다. 이것이 바로 비동기 작업의 완료를 기다리는 다른 부분에 알림을 보내는 메커니즘이다.
- `set_exception(exception)`: `set_result()`와 유사하지만, 정상적인 결과 대신 예외를 설정한다.
- `add_done_callback(callback)`: `Future`가 완료되었을 때 호출될 콜백 함수를 내부 리스트에 추가한다. 이는 `Future`의 완료 이벤트를 구독하는 방법이다. `Future` 상태가 `PENDING`이면 `loop.call_soon(callback, ...)`으로 콜백이 이벤트 루프에서 실행되도록 스케줄링 한다.
- `__await__()`
```Python
def __await__(self):
    if not self.done():
        self._asyncio_future_blocking = True
        yield self  # This tells Task to wait for completion.
    if not self.done():
        raise RuntimeError("await wasn't used with future")
    return self.result()  # May raise too.


__iter__ = __await__  # make compatible with 'yield from'.
```
이 매직 메서드는 `Future`를 awaitable 객체로 만든다. 코루틴이 `Future`를 `await`할 때, 이 메서드가 호출된다. 만약 `Future`가 이미 `FINISHED` 상태라면 즉시 결과나 예외를 반환한다. 만약 아직 `PENDING` 상태라면, `Future` 객체 자신을 `yield`하여 `await`하는 코루틴의 실행을 중단시킨다.
## `Task`
