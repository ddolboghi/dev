---
aliases:
  - CPython Python/ceval.c 분석
---
# 1. 개요
`ceval.c` 파일은 CPython 인터프리터의 심장부로, Python 바이트코드를 실행하는 핵심 로직인 **평가 루프(Evaluation Loop)** 를 포함하고 있다. Python 코드가 실행될 때, 컴파일된 바이트코드는 이 평가 루프에 의해 한 명령어(opcode)씩 해석되고 실행된다. 이 파일의 성능은 Python의 전체 실행 속도에 직접적인 영향을 미친다.

# 2. 주요 구성 요소 및 개념
`ceval.c`의 동작을 이해하기 위한 핵심 요소는 다음과 같다.

## 평가 루프 (`_PyEval_EvalFrameDefault`)
이 파일의 가장 중요한 함수는 `_PyEval_EvalFrameDefault` 다. 이 함수에는 함수의 스택 프레임을 받아 해당 프레임의 바이트코드를 순차적으로 실행하는 거대한 루프가 있다. CPython 3.13에서는 성능 최적화를 위해 전통적인 `switch-case` 문 대신 **Computed goto** 를 사용하여 각 opcode에 해당하는 핸들러로 직접 점프한다.

## 스택 프레임 (`_PyInterpreterFrame`)
스택 프레임은 코드 블록의 실행을 격리하기 위해 고안된 구조다. 예를 들어 함수의 지역 변수를 호출자가 보거나 수정할 수 없어야 하고, 스레드는 다른 스레드의 로컬 데이터에 접근하면 안된다. 이는 스택 프레임 객체 내에서 코드 블록의 실행 컨텍스트를 캡슐화함으로써 구현된다.

Python에서는 지역 객체, 전역 객체, VM이 명령어를 실행하는 동안 데이터를 저장할 스택 같은 것들이 실행 컨텍스트에 포함된다.

그 외에도 스택 프레임은 콜 스택의 구현을 가능하게 한다. 함수를 호출하면 새로운 스택 프레임이 생기고, 호출자의 스택 프레임이 호출된 함수의 스택 프레임에 연결 리스트처럼 연결된다. 이때 현재 실행 중인 함수가 이 연결 리스트의 헤드가 된다. 이 연결 리스트로 현재 스레드의 스택 추적을 생성할 수 있다. 디버거와 프로파일러와 같은 도구가 이러한 방식을 사용한다.

CPython에서 스택 프레임은 Include/internal/pycore_frame.h 파일에 PyInterpreterFrame 구조체로 정의되어있다.
```C
typedef struct _PyInterpreterFrame {
    PyObject *f_executable; /* Strong reference (code object or None) */
    struct _PyInterpreterFrame *previous;
    PyObject *f_funcobj; /* Strong reference. Only valid if not on C stack */
    PyObject *f_globals; /* Borrowed reference. Only valid if not on C stack */
    PyObject *f_builtins; /* Borrowed reference. Only valid if not on C stack */
    PyObject *f_locals; /* Strong reference, may be NULL. Only valid if not on C stack */
    PyFrameObject *frame_obj; /* Strong reference, may be NULL. Only valid if not on C stack */
    _Py_CODEUNIT *instr_ptr; /* Instruction currently executing (or about to begin) */
    int stacktop;  /* Offset of TOS from localsplus  */
    uint16_t return_offset;  /* Only relevant during a function call */
    char owner;
    /* Locals and stack */
    PyObject *localsplus[1];
} _PyInterpreterFrame;
```
- f_executable: 코드 블록이 컴파일된 바이트코드가 포함되어있다.
- previous: 호출자의 스택 프레임을 가리키는 포인터다. 호출자와 호출된 함수의 스택 프레임이 이를 사용하여 연결되어 함수 호출 스택을 생성한다.
- f_globals: 현재 스택 프레임의 컨텍스트 내 모든 전역 객체를 포함하는 딕셔너리다. 런타임은 스택 프레임을 생성하는 동안 이 dictionary를 자동으로 채운다.
- f_builtins: 현재 스택 프레임의 컨텍스트 내 사용 가능한 모든 빌트인을 포함하는 딕셔너리다.
- f_locals: 로컬 변수와 함수 호출 인수가 저장되는 딕셔너리다.
- instr_prt: 현재 실행 중인(또는 곧 시작될) 명령어를 가리키는 포인터다. 모든 스택 프레임에는 실행할 다음 명령어를 가리키는 명령어 포인터가 있다. 이는 스레드의 상태를 유지한다. 예를 들어 VM이 다른 스레드로 전환하는 경우 VM은 실행이 일시 중지된 위치를 알 수 있다.
- stacktop: 스택 상단에 대한 offset이다.
- return_offset: 호출자 프레임의 바이트코드에서 반환할 위치
- localsplus: 로컬 객체를 보관하는 배열로 사용되며 블록의 스택으로도 사용된다. 배열의 첫번째부터 n(로컬 객체 수)번째까지는 로컬 객체를 저장하고 그 뒤의 공간은 스택으로 사용된다. 컴파일러는 로컬 딕셔너리의 해시 기반 조회보다 훨씬 빠른 정수 인덱스를 사용하여 이 배열에 액세스하는, LOAD_FAST와 같은 명령어를 생성하여 로컬 객체에 대한 액세스를 최적화하려고 시도한다.

##
```Python
values = list(range(100))

def binary_search(x):
    return search_in_list(values, x)

def search_in_list(lst, x):
    s = 0
    e = len(lst) - 1

    while s <= e:
        mid = (s + e) // 2
        cmp = compare(x, mid)
        if cmp > 0:
            s = mid + 1
        elif cmp < 0:
            e = mid - 1
        else:
            return mid
    return -1

def compare(a, b):
    traceback.print_stack()
    if a > b:
        return 1
    elif a < b:
        return -1
    return 0
```

위 코드에서 binary_search()를 호출하면 함수 호출 스택은 다음과 같이 생성된다.

```PlainText
compare::stack frame
globals: [values]
locals: [a, b]
previous: search_in_list
---------------------------
search_in_list::stack frame
globals: [values]
locals: [lst, x]
previous: binary_search
--------------------------
binary_search::stack frame
globals: [values]
locals: [x]
previous: __main__
```

## Bytecode & Opcode
Python 소스 코드는 실행 전에 더 낮은 수준의 명령어 집합인 **바이트코드**로 컴파일됩니다. 각 바이트코드 명령어는 **opcode** 와 필요한 경우 **oparg(인자)** 로 구성됩니다.
예를 들어, `LOAD_CONST` 옵코드는 상수 풀에서 특정 상수를 값 스택에 푸시하는 역할을 합니다. 평가 루프는 이 옵코드들을 하나씩 처리하며 프로그램을 실행합니다.

# 3. 실행 흐름
`_PyEval_EvalFrameDefault` 함수의 실행 흐름은 대략 다음과 같습니다.

1.  실행할 프레임을 인자로 받습니다.
2.  무한 루프에 진입하여 프레임의 바이트코드를 실행합니다.
3.  **명령어 포인터**가 가리키는 위치에서 **opcode**와 **oparg**를 가져옵니다.
4.  Computed goto를 사용해 해당 옵코드를 처리하는 코드로 점프합니다.
5.  opcode 핸들러는 스택에서 피연산자를 pop하여 연산을 수행하고, 결과를 다시 스택에 push합니다.
6.  함수 호출(`CALL` 등), 예외 처리, 반복문 등의 제어 흐름을 관리합니다.
7.  함수가 반환(`RETURN_VALUE`)되거나 예외가 발생하여 프레임 실행이 종료되면 루프를 빠져나와 결과 값을 반환합니다.

# 4. CPython 3.13의 주요 특징 및 최적화
`ceval.c`는 CPython의 발전에 따라 지속적으로 개선되어 왔습니다. CPython 3.13 버전에서 관찰되는 주요 특징은 다음과 같습니다.

## 가. Tiered Interpreter
-   **Tier 1 (일반 인터프리터)**: `_PyEval_EvalFrameDefault`가 바로 Tier 1 인터프리터의 구현체입니다. 모든 바이트코드를 실행할 수 있는 범용 실행 엔진입니다.
-   **Tier 2 (최적화 인터프리터)**: 코드 실행 중 'hot' 코드 블록이 식별되면, 해당 코드는 더 빠르고 특화된 명령어(uops)로 구성된 **트레이스(trace)** 로 변환됩니다. 이 트레이스는 `_PyTier2Interpreter` (또는 JIT 컴파일러)에 의해 훨씬 빠르게 실행됩니다. Tier 1보다 훨씬 단순하고 예측 가능성이 높아 월등히 빠릅니다. 이는 반복적인 코드의 실행 속도를 크게 향상시키는 **전문화 적응형 인터프리터(Specializing Adaptive Interpreter)** 의 핵심입니다.

## 나. Inline Caching
속성(attribute) 로드, 전역 변수 접근 등 자주 발생하는 작업의 결과를 캐싱하여 다음 호출 시 더 빠르게 처리합니다. 바이트코드 명령어 자체를 실행 시점에 더 효율적인 버전으로 바꾸는 **적응형 옵코드(adaptive opcode)** 메커니즘을 통해 구현됩니다.

## 5. 결론
`ceval.c`는 Python 코드를 바이트코드 수준에서 실제로 생명을 불어넣는 CPython의 핵심 엔진입니다. 평가 루프, 프레임 관리, 값 스택 연산을 통해 프로그램의 흐름을 제어하며, CPython 3.13에서는 Tiered Interpreter와 같은 고급 최적화 기법을 도입하여 지속적으로 성능을 개선하고 있습니다.
