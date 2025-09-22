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

**여기서부턴 검증 필요**

`Lib/asyncio/tasks.py`에 정의된 `Task` 클래스는 `Future`를 상속받는 특별한 하위 클래스다.

`Task`의 목적은 단 하나의 코루틴을 감싸고, 그 코루틴의 실행을 시작부터 끝까지 책임지고 관리하는 것이다.

사용자가 `asyncio.create_task(coro)`를 호출하면, `Task` 인스턴스가 생성된다. 이때 중요한 점은 코루틴 `coro`가 즉시 실행되지 않는다는 것이다. 대신, `Task`는 자신의 내부 '드라이버' 메서드인 `self.__step()`을 `loop.call_soon()`을 통해 이벤트 루프의 다음 틱에 실행되도록 스케줄링한다. 이로써 `Task`는 이벤트 루프의 스케줄링 주기에 편입되어 코루틴을 구동할 준비를 마친다.

`asyncio`의 협력적 멀티태스킹은 `Task`의 두 비공개 메서드, `__step()`과 `__wakeup()`의 정교한 상호작용을 통해 이루어진다. 이 사이클을 추적하면 `await`가 어떻게 코루틴의 중단과 재개를 유발하는지 명확히 이해할 수 있다.

### 핵심 메커니즘: `__step()` / `__wakeup()` 사이클

```Python
def __step(self, exc=None):
    if self.done(): # 이미 완료된 Task라면 InvalidStateError 일으킴
        raise exceptions.InvalidStateError(
            f'__step(): already done: {self!r}, {exc!r}')
    if self._must_cancel: # 취소해야하는 Task라면 CancelledError 일으킴
        if not isinstance(exc, exceptions.CancelledError):
            exc = self._make_cancelled_error()
        self._must_cancel = False
    self._fut_waiter = None # 기다리는 Future가 없도록 초기화

    _py_enter_task(self._loop, self) # 이벤트 루프의 최근 태스크 목록에 Task 객체를 추가
    try:
        self.__step_run_and_handle_result(exc)
    finally:
        _py_leave_task(self._loop, self)
        self = None  # Needed to break cycles when an exception occurs.
```

`_fut_waiter`는 Task가 현재 `await` 하며 기다리고 있는 Future 객체를 저장하는 역할을 한다.

```Python
def __step_run_and_handle_result(self, exc):
    coro = self._coro
    try:
        if exc is None:
            # We use the `send` method directly, because coroutines
            # don't have `__iter__` and `__next__` methods.
            result = coro.send(None)
        else:
            result = coro.throw(exc)
    except StopIteration as exc:
    ...       
    else:
        blocking = getattr(result, '_asyncio_future_blocking', None)
        if blocking is not None:
            # Yielded Future must come from Future.__iter__().
            if futures._get_loop(result) is not self._loop:
                new_exc = RuntimeError(
                    f'Task {self!r} got Future '
                    f'{result!r} attached to a different loop')
                self._loop.call_soon(
                    self.__step, new_exc, context=self._context)
            elif blocking:
                if result is self:
                    new_exc = RuntimeError(
                        f'Task cannot await on itself: {self!r}')
                    self._loop.call_soon(
                        self.__step, new_exc, context=self._context)

                else: # 코루틴이 다른 비동기 작업을 반환했을때
                    futures.future_add_to_awaited_by(result, self)
                    result._asyncio_future_blocking = False
                    result.add_done_callback( # 코루틴은 비동기 작업이 완료될때까지 대기
                        self.__wakeup, context=self._context)
                    self._fut_waiter = result
                    if self._must_cancel:
                        if self._fut_waiter.cancel(
                                msg=self._cancel_message):
                            self._must_cancel = False
            else:
                new_exc = RuntimeError(
                    f'yield was used instead of yield from '
                    f'in task {self!r} with {result!r}')
                self._loop.call_soon(
                    self.__step, new_exc, context=self._context)

        elif result is None:
            # Bare yield relinquishes control for one event loop iteration.
            self._loop.call_soon(self.__step, context=self._context)
        elif inspect.isgenerator(result):
            # Yielding a generator is just wrong.
            new_exc = RuntimeError(
                f'yield was used instead of yield from for '
                f'generator in task {self!r} with {result!r}')
            self._loop.call_soon(
                self.__step, new_exc, context=self._context)
        else:
            # Yielding something else is an error.
            new_exc = RuntimeError(f'Task got bad yield: {result!r}')
            self._loop.call_soon(
                self.__step, new_exc, context=self._context)
    finally:
        self = None
```

코루틴이 다른 비동기 작업(다른 Task, Future 등)을 await하면, 해당 객체가 반환된다. Task는 이 객체가 완료될 때까지 기다려야 하므로, 반환된 Future-like 객체에 `__wakeup` 메서드를 콜백으로 등록한다. 이 시점에서 `__step` 실행은 종료되고, 제어권은 이벤트 루프에 넘어간다.

`__wakeup`은 단순히 `self.__step()`을 호출하여 Task의 실행을 다시 스케줄링한다. 즉, 기다리던 작업이 끝나면, Task는 다시 실행 큐에 들어가 자신의 다음 단계를 진행할 준비를 한다.

```Python
def __wakeup(self, future):
    futures.future_discard_from_awaited_by(future, self)
    try:
        future.result()
    except BaseException as exc:
        # This may also be a cancellation.
        self.__step(exc)
    else:
        # Don't pass the value of `future.result()` explicitly,
        # as `Future.__iter__` and `Future.__await__` don't need it.
        # If we call `__step(value, None)` instead of `__step()`,
        # Python eval loop would use `.send(value)` method call,
        # instead of `__next__()`, which is slower for futures
        # that return non-generator iterators from their `__iter__`.
        self.__step()
    self = None
```

|행위자|동작|상태 변화|
|---|---|---|
|이벤트 루프|`Task.__step()` 실행|`Task`가 실행 중(running) 상태가 됨|
|`Task.__step_run_and_handle_result()`|`coro.send(None)` 호출. 코루틴은 `await future_obj`를 만날 때까지 실행됨|코루틴이 `future_obj`를 `yield`함|
|`Task.__step_run_and_handle_result()`|`future_obj.add_done_callback(self.__wakeup)` 호출|`Task`는 `future_obj`를 구독하고 일시 중단(paused) 상태가 됨|
|I/O 또는 다른 `Task`|`future_obj.set_result(value)` 호출|`future_obj`가 `FINISHED` 상태가 됨|
|`future_obj`|`loop.call_soon()`을 통해 모든 '완료' 콜백을 스케줄링함|`Task.__wakeup`이 이벤트 루프의 준비 큐에 들어감|
|이벤트 루프|`Task.__wakeup()` 실행|`Task`는 여전히 일시 중단 상태임|
|`Task.__wakeup()`|`loop.call_soon()`을 통해 `Task.__step(result)`를 스케줄링함|`Task.__step`이 이벤트 루프의 준비 큐에 들어감|
|이벤트 루프|`Task.__step(result)` 실행|`Task`가 다시 실행 중 상태가 됨. 1단계부터 사이클 반복|

이 사이클을 소스 코드 관점에서 더 자세히 살펴보자.

- `Task.__step(exc=None)`:
    1. 이 메서드는 코루틴의 운전자다. 이벤트 루프에 의해 처음 호출되거나, `__wakeup`에 의해 다시 스케줄링되어 호출된다.
    2. `coro.send(result)` 또는 `coro.throw(exc)`를 호출하여 코루틴을 다음 `yield` 지점(코루틴 입장에서는 `await` 지점)까지 진행시킨다.
    3. 코루틴은 `await obj` 표현식을 만나 실행을 멈춘다. 바이트코드 수준에서는 YIELD_VALUE가 실행되고, `obj.__await__()`가 반환한 이터레이터(보통 `Future` 객체 자체, `F_wait`라 칭함)가 `__step` 메서드로 반환된다.
    4. `__step` 메서드는 반환받은 `F_wait`가 아직 완료되지 않았음을 확인하고, `F_wait.add_done_callback(self.__wakeup)`를 호출한다. 이는 Task가 Future의 완료를 기다리기 위해 자신을 등록하는, 즉 '구독'하는 것이라고 볼 수 있다.
    5. 콜백 등록 후 `__step` 메서드는 종료된다. 이 시점에서 `Task`는 `F_wait`의 완료를 기다리며 '일시 중단' 상태가 된다. 이벤트 루프는 이제 다른 준비된 `Task`를 실행할 수 있다.
- `Future.set_result(value)`:
    1. 시간이 흘러, 다른 코드가 비동기 작업을 완료하고 `F_wait.set_result(value)`를 호출한다.
    2. `set_result`는 `F_wait`의 상태를 `FINISHED`로 변경하고, 결과값을 저장한 뒤, 등록된 모든 완료 콜백을 순회한다.
    3. 여기서 `Task`가 등록해 둔 `self.__wakeup` 메서드를 발견하고, `loop.call_soon(self.__wakeup, F_wait)`를 통해 이벤트 루프에 이 콜백의 실행을 요청한다.
- `Task.__wakeup(future)`:
    1. 이벤트 루프의 다음 틱에서, 스케줄링되었던 `__wakeup` 메서드가 실행된다.
    2. `self.__step()`을 호출하여 Task의 실행을 다시 스케줄링한다.
    3. `__step`이 다시 실행될 때, `F_wait`는 이미 완료되었으므로 그 결과값을 가져와 `coro.send()`를 통해 일시 중단되었던 코루틴 내부로 전달한다. 코루틴은 `await` 표현식 다음 줄부터 실행을 재개한다.

`await` 키워드는 `Task`가 `Future`의 완료 이벤트를 구독하도록 하는 문법적 신호에 불과하다. 이 메커니즘은 결과의 생산자(예: 네트워크 프로토콜)와 소비자(코루틴)를 우아하게 분리한다. 생산자는 `Future` 객체의 존재만 알면 되며, 작업이 완료되면 `future.set_result()`를 호출하기만 하면 된다. 소비자인 `Task`나 코루틴에 대해 전혀 알 필요가 없다. `Future`는 익명의 일회성 통신 채널 역할을 하여, 확장성 높고 결합도가 낮은 시스템을 가능하게 한다.

## `asyncio.run()` 실행 과정 추적

### 진입점: `asyncio.run()`

Python 3.7부터 `asyncio` 애플리케이션을 실행하는 권장 방법은 `Lib/asyncio/runners.py`에 위치한 `asyncio.run()` 함수를 사용하는 것이다. 이 고수준 함수는 이벤트 루프의 생성, 실행, 그리고 자원 정리에 이르는 모든 과정을 편리하게 처리해준다.

`asyncio.run()`의 소스 코드를 분석하면 다음과 같은 핵심적인 설정 및 해제 단계를 수행함을 알 수 있다.
```Python
def run(main, *, debug=None, loop_factory=None):
    if events._get_running_loop() is not None:
        # fail fast with short traceback
        raise RuntimeError(
            "asyncio.run() cannot be called from a running event loop")
    
    with Runner(debug=debug, loop_factory=loop_factory) as runner:
        return runner.run(main)
```
  

```Python
class Runner:
  def run(self, coro, *, context=None):
      """Run code in the embedded event loop."""
      if not coroutines.iscoroutine(coro):
          raise ValueError("a coroutine was expected, got {!r}".format(coro))
      
      if events._get_running_loop() is not None:
          # fail fast with short traceback
          raise RuntimeError(
              "Runner.run() cannot be called from a running event loop")
  
      self._lazy_init() # 이벤트 루프 생성 및 Runner 초기화
  
      if context is None:
          context = self._context
  
      task = self._loop.create_task(coro, context=context) # 코루틴 객체를 Task 객체로 만듬
      # ...
      
      try:
          return self._loop.run_until_complete(task) # Task 객체가 완료될때까지 기다림
      except exceptions.CancelledError:
          if self._interrupt_count > 0:
              uncancel = getattr(task, "uncancel", None)
              if uncancel is not None and uncancel() == 0:
                  raise KeyboardInterrupt()
          raise  # CancelledError
      finally:
          if (sigint_handler is not None
              and signal.getsignal(signal.SIGINT) is sigint_handler
          ):
              signal.signal(signal.SIGINT, signal.default_int_handler)
              
  def _lazy_init(self):
      if self._state is _State.CLOSED:
          raise RuntimeError("Runner is closed")
      if self._state is _State.INITIALIZED:
          return
      if self._loop_factory is None:
          self._loop = events.new_event_loop() # 새 이벤트 루프 생성
          if not self._set_event_loop:
              # Call set_event_loop only once to avoid calling
              # attach_loop multiple times on child watchers
              events.set_event_loop(self._loop) # 현재 스레드의 활성 이벤트 루프로 설정
              self._set_event_loop = True
      else:
          self._loop = self._loop_factory()
      if self._debug is not None:
          self._loop.set_debug(self._debug)
      self._context = contextvars.copy_context()
      self._state = _State.INITIALIZED
            
```
1. 루프 중첩 방지: 현재 스레드에서 이미 다른 이벤트 루프가 실행 중인지 확인하여, `asyncio.run()`의 중첩 호출로 인한 `RuntimeError`를 방지한다.
2. 새 이벤트 루프 생성: `events.new_event_loop()`를 호출하여 현재 플랫폼에 맞는 새로운 이벤트 루프 인스턴스(예: `SelectorEventLoop`)를 생성한다.
3. 현재 루프 설정: 생성된 새 루프를 `events.set_event_loop()`를 통해 현재 스레드의 활성 이벤트 루프로 설정한다.
4. 메인 태스크 생성: 사용자로부터 전달받은 메인 코루틴을 `loop.create_task(main_coro)`를 사용하여 `Task` 객체로 감싼다. 이 과정에서 `Task`의 `__step()` 메서드가 `loop.call_soon()`으로 스케줄링된다.
5. 루프 실행: `loop.run_until_complete(main_task)`를 호출하여 이벤트 루프를 시작한다. 이 메서드는 `main_task`가 완료될 때까지 `run_forever()`를 내부적으로 실행한다.
6. 자원 정리 (finally 블록): 모든 실행 로직은 `try...finally` 블록으로 감싸져 있다. 사용자의 코드가 정상적으로 종료되거나 예외를 발생시키더라도, `finally` 블록은 항상 실행되어 다음의 중요한 정리 작업을 보장한다 :

- 기본 스레드 풀 실행기(executor)와 같은 관련 리소드를 안전하게 종료한다.
- `loop.close()`를 호출하여 이벤트 루프와 관련된 모든 큐, 셀렉터, 소켓 등의 자원을 완전히 해제한다.

이러한 구조 덕분에 개발자는 이벤트 루프의 생명주기를 수동으로 관리하는 복잡함 없이 비동기 코드 실행에만 집중할 수 있다.

### 실행 추적: Hello... World!

이제 `asyncio`의 표준 "Hello World" 예제가 `asyncio.run()`을 통해 어떻게 실행되는지 단계별로 추적해 보자. 이 과정을 통해 앞서 분석한 모든 개념이 실제로 어떻게 맞물려 돌아가는지 확인할 수 있다.

```Python
import asyncio
import time

async def main():
    print(f"[{time.time():.2f}] Hello...")
    await asyncio.sleep(1)
    print(f"[{time.time():.2f}]... World!")

asyncio.run(main())
```

실행 과정 추적:

1. `asyncio.run(main())` 호출: `runners.py`의 `run` 함수가 실행된다. 새로운 `SelectorEventLoop`가 생성되고 현재 스레드의 이벤트 루프로 설정된다.
2. 태스크 생성: `main` 코루틴 객체가 `main_task = loop.create_task(main())`를 통해 `Task`로 감싸진다. `main_task.__step()`이 `loop.call_soon()`으로 스케줄링되어 루프의 `_ready` 큐에 추가된다.
3. 루프 시작: `loop.run_until_complete(main_task)`가 호출되어 이벤트 루프의 `run_forever()`가 시작된다.
4. 첫 번째 틱(Tick 1): 이벤트 루프는 `_ready` 큐에서 `main_task.__step()`을 꺼내 실행한다.
    - `main_task`의 `main` 코루틴이 `send(None)`으로 시작된다.
    - `"Hello..."`가 출력된다.
    - 코루틴은 `await asyncio.sleep(1)` 라인을 만난다.
5. `asyncio.sleep(1)` 내부 동작:
    - `asyncio.sleep(1)` 코루틴은 내부적으로 현재 루프를 가져와 새로운 `Future` 객체(`sleep_future`)를 생성한다.
    - `loop.call_later(1, sleep_future.set_result, None)`를 호출하여 1초 후에 `sleep_future`를 완료시키는 콜백을 스케줄링한다.
    - `asyncio.sleep(1)`은 이 `sleep_future`를 반환한다.
6. 태스크 중단: `main` 코루틴은 `sleep_future`를 `await`한다. `main_task.__step()`은 `yield`된 `sleep_future`를 받고, `sleep_future.add_done_callback(main_task.__wakeup)`를 통해 자신을 `sleep_future`의 완료 콜백으로 등록한다. 그 후 `__step`은 종료되고, `main_task`는 `sleep_future`를 기다리며 일시 중단 상태가 된다.
7. OS 대기: 이벤트 루프는 `_ready` 큐가 비어있는 것을 확인한다. 스케줄링된 작업 중 가장 빠른 것이 1초 후의 타이머이므로, `self._selector.select(1.0)`을 호출한다. Python 프로세스는 OS 커널에 의해 약 1초간 휴면 상태에 들어간다.
8. 타이머 만료 (약 1초 후): `select()`의 타임아웃이 만료되어 반환된다. 이벤트 루프는 스케줄링된 타이머를 확인하고, `sleep_future.set_result(None)` 콜백이 실행될 시간이 되었음을 인지하여 `_ready` 큐에 넣는다.
9. 두 번째 틱(Tick 2): 이벤트 루프는 `_ready` 큐에서 `sleep_future.set_result(None)` 콜백을 꺼내 실행한다.
10. 퓨처 완료 및 콜백 스케줄링: `sleep_future`의 상태가 `FINISHED`로 변경된다. 이어서 등록된 완료 콜백인 `main_task.__wakeup`이 `loop.call_soon()`을 통해 `_ready` 큐에 추가된다.
11. 세 번째 틱(Tick 3): 이벤트 루프는 `_ready` 큐에서 `main_task.__wakeup`을 꺼내 실행한다.
12. 태스크 재시작 준비: `__wakeup` 메서드는 `main_task.__step`을 다시 `loop.call_soon()`으로 스케줄링하여 `_ready` 큐에 넣는다.
13. 네 번째 틱(Tick 4): 이벤트 루프는 `_ready` 큐에서 `main_task.__step`을 꺼내 실행한다.
    - `__step`은 완료된 `sleep_future`의 결과(`None`)를 `main` 코루틴에 `send()`한다.
    - `main` 코루틴은 `await` 지점 다음부터 실행을 재개하여 `"... World!"`를 출력한다.
    - 코루틴의 끝에 도달하여 `StopIteration` 예외를 발생시키며 종료된다.
14. 태스크 완료: `main_task.__step`은 `StopIteration`을 포착하고, `Task` 자체(이 또한 `Future`이므로)의 결과를 `set_result()`를 통해 설정한다. `main_task`는 이제 `FINISHED` 상태가 된다.
15. 루프 종료: `run_until_complete(main_task)`는 `main_task`가 완료된 것을 확인하고, 이벤트 루프의 `_stopping` 플래그를 설정하여 `run_forever()` 루프를 탈출한다.
16. 자원 정리 및 종료: `asyncio.run()`의 `finally` 블록이 실행되어 `loop.close()`를 호출하고 모든 자원을 정리한다. 프로그램이 정상적으로 종료된다.

이 전체 실행 과정을 통해 `asyncio` 시스템의 각 구성 요소가 맡은 역할이 명확해진다. `async`/`await` 구문은 코루틴의 상태를 관리하고, `asyncio.sleep`과 같은 라이브러리 함수는 시간 기반 이벤트에 대한 `Future` 핸들을 제공하며, `Task`는 이 `Future`의 완료를 구독하는 다리 역할을 한다. 마지막으로 이벤트 루프는 시계이자 콜백 실행기로서 이 모든 것을 조율한다. 동시성이라는 복잡한 동작은 어느 한 구성 요소의 독단적인 기능이 아니라, 이들의 정밀하게 정의된 상호작용으로부터 창발되는 것이다.
