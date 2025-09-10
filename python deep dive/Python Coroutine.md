---
aliases:
  - async/await
---
단순한 제너레이터(Generator)에서 시작하여, 양방향 통신 채널로의 발전, 그리고 `yield from`을 통한 위임(Delegation) 메커니즘의 도입 과정을 추적한다. 이를 통해 `await` 키워드가 완전히 새로운 개념이 아니라, 제너레이터 기반 코루틴의 실행 모델을 계승하고 구문적으로 명확화한 결과물임을 바이트코드 수준에서 증명한다.
# Python 코루틴의 기원: 제너레이터와 위임
함수 정의 내에 `yield` 키워드가 포함되면, 그 함수는 더 이상 일반적인 서브루틴(Subroutine)이 아니라 제너레이터 함수(Generator Function)로 취급된다. 제너레이터 함수를 호출하면 함수 본문이 즉시 실행되지 않고, 대신 제너레이터 객체(Generator Object)라는 특별한 종류의 이터레이터를 반환한다.
## Generator
제너레이터 객체는 상태를 가지는 이터레이터고, 실행을 중단하고 재개할 수 있다. 제너레이터 객체의 `__next__()` 메서드가 호출될 때마다 함수 본문은 `yield`를 만날 때까지 실행된다. `yield`에 도달하면, 제너레이터는 `yield` 뒤의 표현식 값을 호출자에게 반환하고 자신의 실행을 그 자리에서 '동결(freeze)'시킨다. 이 동결 상태에는 지역 변수의 현재 바인딩, 명령어 포인터(Instruction Pointer), 내부 평가 스택(Evaluation Stack) 등 모든 지역 상태 정보가 포함된다. 이후 다시 `__next__()`가 호출되면, 제너레이터는 저장된 상태를 복원하고 중단되었던 지점 바로 다음부터 실행을 재개한다.

비동기 프로그래밍 관점에서, 이는 하나의 함수가 자신의 실행 흐름을 자발적으로 양보하고, 나중에 다시 제어권을 넘겨받아 작업을 이어갈 수 있게 하는 협력적 멀티태스킹의 가장 원시적인 형태를 제공한다.
## yield
Python 2.5에서 `yield`가 문(Statement)에서 표현식(Expression)으로 변경되면서 제너레이터는 값을 생산할 뿐만 아니라, 외부로부터 값을 수신할 수 있는 양방향 통신 능력을 갖게 되었다.

```Python
import itertools

def averager():
    """A coroutine that computes a running average."""
    total = 0.0
    count = 0
    average = None
    while True:
        term = (yield average)  # Receive a value and yield the current average
        total += term
        count += 1
        average = total / count

# Coroutine driver
coro = averager()
next(coro)  # Prime the coroutine by advancing to the first yield
print(coro.send(10))  # Sends 10, yields 10.0 print 10.0
print(coro.send(20))  # Sends 20, yields 15.0 print 15.0
print(coro.send(30))  # Sends 30, yields 20.0 print 20.0
```
위 예제에서 `term = (yield average)` 라인은 제너레이터의 양방향 통신 능력을 명확히 보여준다. `yield average`는 현재 `average`값을 호출자에게 반환하며 실행을 중단하고, 이후 호출자가 `.send(term)`을 호출하면 중단되었던 `yield` 표현식이 `term` 값을 반환하며 실행이 재개된다. `next()`는 내부적으로 `send(None)`과 같다.

즉, 호출자가 `.send(value)`를 호출하면, 중단되어 있던 제너레이터가 재개되면서 `yield` 표현식 자체가 `value`를 반환한다.

## yield from
Python 3.3에서 `yield from`표현식이 도입되어 제너레이터가 제어권을 하위 제너레이터에게 완전히 위임할 수 있게 되었다. `yield from <subgenerator>` 표현식은 최상위 호출자와 하위 제너레이터가 다음과 같이 작동하도록 한다.
1. 하위 제너레이터가 `yield`하는 모든 값은 중간 제너레이터를 거치지 않고 최상위 호출자에게 직접 전달된다.
2. 최상위 호출자가 `.send()`로 보낸 값은 하위 제너레이터의 `yield` 표현식으로 직접 전달된다.
3. 최상위 호출자가 `.throw()`로 주입한 예외는 하위 제너레이터 내에서 발생한다.
4. 하위 제너레이터가 `return` 문을 통해 종료될 때, 그 반환값은 상위 제너레이터에서 `yield from` 표현식의 결과값이 된다. 이는 `StopIteration` 예외의 `value` 속성을 통해 구현된다.  

```Python
# 3. 가장 안쪽의 하위 제너레이터
def subgenerator():
    """가장 안쪽에서 실제 작업을 처리하는 제너레이터"""
    print("      [하위] 시작")
    total = 0
    try:
        while True:
            # 1-2. 호출자의 send() 값을 여기서 받음
            received = yield total 
            print(f"      [하위] '{received}' 받음")
            if received is None:
                break
            total += received
    except ValueError as e:
        # 2-2. 호출자의 throw() 예외를 여기서 처리
        print(f"      [하위] 예외 처리: {e}")
    finally:
        print("      [하위] 종료")
        # 3-2. 최종 결과를 반환
        return f"최종 합계: {total}"

# 2. 중간 제너레이터
def middle_generator():
    """yield from을 사용하여 하위 제너레이터에 제어권을 위임"""
    print("   [중간] 시작")
    # yield from을 통해 subgenerator와 양방향 채널 연결
    # subgenerator가 종료되면 그 반환값이 result에 저장됨
    result = yield from subgenerator() 
    print(f"   [중간] 하위 제너레이터 결과: '{result}'")
    print("   [중간] 종료")
    return result # 결과를 다시 호출자에게 전달

# 1. 최상위 호출자 (main 로직)
def main():
    """제너레이터를 제어하고 상호작용하는 최상위 호출자"""
    print("[호출자] 시작")
    gen = middle_generator()

    # 1-1. 제너레이터 첫 실행 및 값 전달
    # next()는 내부적으로 send(None)과 같음. subgenerator의 yield total을 실행.
    initial_value = next(gen) 
    print(f"[호출자] 초기 값 '{initial_value}' 받음\n")

    # 1-2. .send()로 값 보내기
    # 이 값은 middle_generator를 거치지 않고 subgenerator의 'received = yield total'로 바로 전달됨.
    value_after_send_10 = gen.send(10)
    print(f"[호출자] 10 보낸 후 '{value_after_send_10}' 받음\n")

    value_after_send_20 = gen.send(20)
    print(f"[호출자] 20 보낸 후 '{value_after_send_20}' 받음\n")

    # 2-1. .throw()로 예외 주입하기
    # 이 예외는 subgenerator 내부의 try...except 블록에서 잡힘.
    try:
        print("[호출자] ValueError 예외 주입 시도")
        gen.throw(ValueError("테스트 예외"))
    except StopIteration as e:
        # subgenerator가 예외 처리 후 finally를 거쳐 return하면, 
        # 호출자 입장에서는 StopIteration이 발생하며 그 반환값이 e.value에 담김.
        print(f"[호출자] StopIteration 잡음. 최종 결과: '{e.value}'")

    print("[호출자] 종료")

if __name__ == "__main__":
    main()
```

`Python/ceval.c` 파일의 `PyEval_EvalFrameEx` 함수는 파이썬 바이트코드를 하나씩 해석하고 실행하는 역할을 한다.
`yield from` 표현식은 `YIELD_FROM`이라는 바이트코드 옵코드(opcode)로 컴파일된다.
바이트코드는 스택 기반 인터프리터인 PVM에서 실행된다.
`PyEval_EvalFrameEx` 내의 `YIELD_FROM` 옵코드 처리 로직을 분석하면 다음과 같은 동작을 확인할 수 있다:
1. 스택의 최상단(TOS, Top of Stack)에서 위임할 이터레이터(하위 제너레이터)를 가져온다.
2. 스택의 두 번째 값(TOS1)을 가져와 하위 제너레이터의 `.send()` 메서드로 전달한다. (첫 시작 시에는 `None`이 전달된다.)
3. `.send()` 호출의 결과를 확인한다.
    - 만약 `StopIteration` 예외가 발생하면, 위임이 끝났음을 의미한다. 예외 객체의 `value` 속성을 스택에 푸시하여 `yield from` 표현식의 결과값으로 삼고, 상위 제너레이터의 실행을 계속한다.
    - 만약 다른 값이 반환되면 (즉, 하위 제너레이터가 `yield`한 값), 이 값을 스택에 푸시하고 상위 제너레이터의 실행을 일시 중단한다. 이 값은 최상위 호출자에게 전달된다.
    - 다른 예외가 발생하면, 해당 예외를 상위 제너레이터로 전파한다.

`yield from`의 C 레벨 구현은 최상위 호출자와 하위 제너레이터 간의 통신을 효율적으로 중개해준다.

`yield`는 중단점을 제공했고, `.send()`는 양방향 통신을 가능하게 했으며, `yield from`은 조합과 위임을 통해 복잡성을 해결했다.

# async/await
제너레이터 기반 코루틴과 `yield from`은 Python에서 비동기 프로그래밍을 가능하게 했지만, 동일한 `yield` 키워드가 이터레이션을 위한 데이터 생성과 비동기 제어 흐름을 위한 중단 신호라는 두 가지 상이한 목적으로 사용되어 코드의 가독성을 해치고 초보자에게 혼란을 야기했다.
## Generator와 Coroutine 분리
`async def`로 선언된 함수는 '네이티브 코루틴 함수'로 정의된다. 이 함수는 호출 시 항상 코루틴 객체를 반환하며, 내부에 `await` 표현식이 없더라도 코루틴으로 취급된다.

`async def` 함수 내에서 `yield`나 `yield from`을 사용하면 `SyntaxError`가 발생한다. 이로써 개발자는 함수의 시그니처만 보고도 이 함수가 데이터를 생성하는 제너레이터인지, 아니면 비동기 작업을 수행하는 코루틴인지를 명확하게 구분할 수 있게 되었다.
## `await`는 특화된 `yield from`이다
`async`/`await`는 기존 제너레이터 실행 모델 위에 구축된 정교한 문법적 추상화 계층이다.

| 제너레이터 기반 코루틴 (`yield from`)                                                                                                                                        | 네이티브 코루틴 (`await`)                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `python import types @types.coroutine def old_coro(awaitable): val = yield from awaitable return val`                                                              | `python async def new_coro(awaitable): val = await awaitable return val`                                                                                     |
| `dis.dis(old_coro)` 출력 (Python 3.7 기준)                                                                                                                             | `dis.dis(new_coro)` 출력 (Python 3.7 기준)                                                                                                                       |
| 4 0 LOAD_FAST 0 (awaitable)<br>2 GET_YIELD_FROM_ITER<br>4 LOAD_CONST 0 (None)<br>6 YIELD_FROM<br>8 STORE_FAST 1 (val) 5<br>10 LOAD_FAST 1 (val)<br>12 RETURN_VALUE | 4 0 LOAD_FAST 0 (awaitable)<br>2 GET_AWAITABLE<br>4 LOAD_CONST 0 (None)<br>6 YIELD_FROM<br>8 STORE_FAST 1 (val) 5<br>10 LOAD_FAST 1 (val)<br>12 RETURN_VALUE |
|                                                                                                                                                                    |                                                                                                                                                              |
- 두 버전 모두 **코루틴의 실행을 중단하고 제어권을 위임하기 위해 `YIELD_FROM` 옵코드를 사용**한다. 이는 `await`가 `yield from`과 동일한 저수준 위임 메커니즘에 의존하고 있음을 보여준다.
- 네이티브 코루틴의 바이트코드에는 `GET_AWAITABLE`이라는 옵코드를 사용하여 Awaitable 객체에만 `await`를 사용할 수 있도록 제한한다.
- `await`는 `YIELD_FROM` 중단 메커니즘을 처리할 수 있는 특별한 프레임 상태를 가진 `async def` 함수 내에서만 호출될 수 있기 때문에, 비동기 호출 체인은 끊어질 수 없다.
## op_code: `GET_AWAITABLE`
GET_AWAITABLE 옵코드는 스택 최상단에 있는 객체(피연산자)가 'Awaitable'인지 확인하고, 만약 그렇다면 `__await__` 메서드를 호출하여 반환된 이터레이터를 스택에 다시 푸시한다.

PEP 492에 따르면, 객체가 어웨이터블이 되기 위한 조건은 다음 중 하나이다.
- 네이티브 코루틴 객체일 경우
- `@types.coroutine`으로 데코레이팅된 제너레이터 기반 코루틴 객체일 경우
- 이터레이터를 반환하는 `__await__` 매직 메서드를 가진 객체일 경우

**`asyncio.Future` 객체가 `await` 가능하게 된 이유는 단순히 `Future` 클래스에 `__await__ = __iter__` 한 줄을 추가했기 때문이다.**
`GET_AWAITABLE` 옵코드는 `__await__` 프로토콜을 호출하여 `Future` 객체 자체(이터레이터이므로)를 `YIELD_FROM`이 처리할 수 있도록 준비시킨다. 즉, 이 옵코드는 `await`가 비동기 제어 흐름을 위한 것임을 바이트코드 수준에서 보장한다.
## CPython의 PyCoroObject 구조체
CPython 내부에서 네이티브 코루틴은 제너레이터와는 다른 별개의 C 구조체로 표현된다. `Objects/genobject.c`에 정의된 `PyCoroObject` 구조체가 바로 그것이다. 이 구조체는 코루틴의 실행 프레임(`cr_frame`), 실행 상태(`cr_running`), 코드 객체(`cr_code`) 등에 대한 정보를 담고 있다.

중요한 점은 `PyCoroObject`가 C-API 수준에서 `__iter__`와 `__next__` 메서드를 구현하지 않는다는 것이다. 따라서 네이티브 코루틴 객체를`iter()` 함수에 전달하거나 일반 `for` 루프에서 사용하려고 하면 `TypeError`가 발생한다. 이는 제너레이터와 코루틴의 개념적 분리를 인터프리터의 가장 낮은 수준에서부터 강제하는 설계적 결정이다.


