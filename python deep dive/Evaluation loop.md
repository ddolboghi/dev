---
aliases:
  - CPython Python/ceval.c 분석
---
# 1. 개요
`ceval.c` 파일은 CPython 인터프리터의 심장부로, Python 바이트코드를 실행하는 핵심 로직인 **평가 루프(Evaluation Loop)** 를 포함하고 있다. Python 코드가 실행될 때, 컴파일된 바이트코드는 이 평가 루프에 의해 한 명령어(opcode)씩 해석되고 실행된다. 이 파일의 성능은 Python의 전체 실행 속도에 직접적인 영향을 미친다.

# 2. 주요 구성 요소 및 동작 원리
평가 루프의 동작을 이해하기 위한 핵심 요소는 다음과 같다.
## Bytecode & Opcode
각 바이트코드 명령어는 opcode와 oparg(인자), 2byte로 구성된다. 명령어가 인자를 필요로하지 않으면 바이트코드의 인자 값은 0이 된다.
실제 모든 명령어는 Python/bytecodes.c에 정의되어있다.

opcode는 Include/opcode_ids.h에 정의되어있다. 각 명령어마다 opcode id가 부여된다.
```h
#define CACHE                                    0
#define BINARY_SLICE                             1
#define BUILD_TEMPLATE                           2
#define BINARY_OP_INPLACE_ADD_UNICODE            3
#define CALL_FUNCTION_EX                         4
#define CHECK_EG_MATCH                           5
#define CHECK_EXC_MATCH                          6
#define CLEANUP_THROW                            7
#define DELETE_SUBSCR                            8
#define END_FOR                                  9
#define END_SEND                                10
...
```

> [!tip]
    > CACHE는 컴파일러가 성능 향상을 목적으로 명령어를 최적화하기 위해 추가해주는 opcode다.

만약 인자 값이 1byte보다 크다면(255이상) 바이트 코드에 할당할 수 없다. 
컴파일러는 인자 값을 1byte에 할당할 수 없을때 EXTENDED_ARG 명령어를 사용하여 여분의 byte를 가져온다.
하나의 명령어 앞에 최대 3개의 연속된 EXTENDED_ARG 명령어를 붙일 수 있다. 이는 명령어의 최대 인자 크기가 4byte(명령어 자체 인자의 1byte, 3개의 EXTENDED_ARG 명령어를 통해 가져온 3byte)로 제한된다는 것을 의미한다.
### 바이트코드 실행
바이트코드 인터프리터의 엔진은 ceval.c에 구현되있는 거대한 루프다. Python/ceval.c 파일의 _PyEval_EvalFrameDefault 함수로 구현된다.

```C
PyObject* _Py_HOT_FUNCTION DONT_SLP_VECTORIZE
_PyEval_EvalFrameDefault(PyThreadState *tstate, _PyInterpreterFrame *frame, int throwflag)
```

- tstate는 인터프리터에서 코드를 실행하는 스레드의 상태를 가리키는 포인터다.
- frame은 인터프리터에서 실행될 Python 코드 블록의 스택프레임을 가리키는 포인터다.

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

### 스택 프레임 예시
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

## 평가 루프 (`_PyEval_EvalFrameDefault`)
평가 루프는 다음 명령어를 실행하는 디스패치 루프이기도 하다. 이 루프는 Python/ceval.c 파일에 정의된 `_PyEval_EvalFrameDefault()` 함수에 의해 작동한다.
### 준비
```C
PyObject* _Py_HOT_FUNCTION DONT_SLP_VECTORIZE
_PyEval_EvalFrameDefault(PyThreadState *tstate, _PyInterpreterFrame *frame, int throwflag)
{
    _Py_EnsureTstateNotNULL(tstate);
    CALL_STAT_INC(pyeval_calls);

#if USE_COMPUTED_GOTOS
/* Import the static jump table */
#include "opcode_targets.h"
#endif

#ifdef Py_STATS
    int lastopcode = 0;
#endif
    uint8_t opcode;    /* Current opcode */
    int oparg;         /* Current opcode argument, if any */

...

    /* 루프를 위한 준비 */
    _PyInterpreterFrame  entry_frame;

    entry_frame.f_executable = Py_None;
    entry_frame.instr_ptr = (_Py_CODEUNIT *)_Py_INTERPRETER_TRAMPOLINE_INSTRUCTIONS + 1;
    entry_frame.stacktop = 0;
    entry_frame.owner = FRAME_OWNED_BY_CSTACK;
    entry_frame.return_offset = 0;
    /* Push frame */
    entry_frame.previous = tstate->current_frame;
    frame->previous = &entry_frame;
    tstate->current_frame = frame;

...

    /* Local "register" variables.
     * These are cached values from the frame and code object.  */
    _Py_CODEUNIT *next_instr;
    PyObject **stack_pointer;

    
#if defined(_Py_TIER2) && !defined(_Py_JIT)
    /* Tier 2 interpreter state */
    _PyExecutorObject *current_executor = NULL;
    const _PyUOpInstruction *next_uop = NULL;
#endif

start_frame:
    if (_Py_EnterRecursivePy(tstate)) {
        goto exit_unwind;
    }

    next_instr = frame->instr_ptr;
resume_frame:
    stack_pointer = _PyFrame_GetStackPointer(frame);
...
}    

```
- opcode와 oparg 변수는 다음 바이트코드 명령어와 그 인자 값을 저장한다. 이 변수는 루프를 반복할 때마다 계속 업데이트된다.
- entry_frame 객체가 초기화되고 실행될 프레임의 이전 프레임으로 설정된다. 이렇게 하면 entry_frame이 인터프리터에서 최하단 프레임이 되고, 모든 파이썬 코드가 실행을 완료하면 마지막에 실행된다. 여기에는 PVM에 루프를 종료하도록 지시하는 바이트코드 명령어가 포함되어 있다.
- next_instr은 다음 명령을 가리키는 명령어 포인터다. 현재 프레임의 명령어 포인터 값으로 초기화된다. VM이 실행 프레임을 변경하면(e.g. 함수 호출) 이 변수 역시 새 프레임의 명령어 포인터 값으로 바뀐다.

### 루프 본문
준비 후 매크로 사용에 크게 의존하는 디스패치 루프 본문이 시작된다.

컴파일러가 computed goto를 지원하지 않으면 switch 케이스, 지원하면 computed goto에 대한 코드를 생성할 수 있는 전처리기 매크로를 사용한다. 이러한 매크로는 루프의 한 번의 반복을 위한 코드로 집합적으로 확장된다.

매 반복마다 다음과 같은 일이 일어난다:
1. `NEXTOPARG()` 매크로를 사용하여 다음에 실행할 명령어에 대한 opcode를 가져온다.
2. `DISPATCH_GOTO()` 매크로를 사용하여 이를 실행할 opcode 구현체로 점프한다. 만약 computed goto가 사용 중이지 않으면 switch 블록의 시작점으로 점프한다. 하지만 computed goto가 사용 중이면, opcode_targets 점프 테이블의 조회로 확장되어 opcode 구현으로 점프한다. 이 매크로는 Python/ceval_macros.h에 정의되어있다.

```h
#if USE_COMPUTED_GOTOS
#  define TARGET(op) TARGET_##op:
#  define DISPATCH_GOTO() goto *opcode_targets[opcode]
#else
#  define TARGET(op) case op: TARGET_##op:
#  define DISPATCH_GOTO() goto dispatch_opcode
#endif
```
`TARGET()` 매크로는 opcode 구현을 실행하기 위해 사용된다.

이 두 동작은 Python/ceval_macros.h 파일의 `DISPATCH()` 매크로 안에 함께 들어있다. `DISPATCH()` 매크로를 호출하면 다음 opcode를 가져와 실행한다.
```h
#define DISPATCH() \
    { \
        NEXTOPARG(); \
        PRE_DISPATCH_GOTO(); \
        DISPATCH_GOTO(); \
    }
```
`PRE_DISPATCH_GOTO()` 매크로는  Python이 특수한 디버깅 모드로 실행될때 저수준 추적을 수행한다. CPython 개발을 위해 사용되므로 일반적인 Python 프로그램에서는 작동하지 않는다.

종합하면 루프는 다음 순서로 동작한다.
```C
DISPATCH();

    {
    /* Start instructions */
#if !USE_COMPUTED_GOTOS
    dispatch_opcode:
        switch (opcode)
#endif
        {

#include "generated_cases.c.h"
...
    {
    ...    
     DISPATCH_GOTO()
    }    
}
```
1. `DISPATCH()`를 통해 첫 명령어의 opcode를 가져오고 그것의 구현체로 점프한다.
2. computed goto가 사용 중이지 않으면 `dispatch_opcode` 레이블을 시작점으로 사용하는 switch case 코드가 생성된다.
3. `#include "generated_cases.c.h"`로 모든 opcode 구현 코드를 불러온다. 이 파일에는 실제 모든 opcode를 처리하는 C 코드가 들어있다. 이 파일은 CPython 빌드 시점에 자동으로 생성되며, 각 옵코드를 처리하는 `TARGET(OPCODE_NAME): { ... }` 형태의 코드 블록들로 가득 차 있다.
4. generated_cases.c.h의 opcode 구현코드는 매크로를 통해 코드가 바뀐다.
	```C
	TARGET(LOAD_FAST) {
	    frame->instr_ptr = next_instr;
	    next_instr += 1;
	    INSTRUCTION_STATS(LOAD_FAST);
	    PyObject *value;
	    value = GETLOCAL(oparg);
	    assert(value != NULL);
	    Py_INCREF(value);
	    stack_pointer[0] = value;
	    stack_pointer += 1;
	    DISPATCH();
	}
	```

5. computed goto를 사용할때:
	```C
	TARGET_LOAD_FAST: {
	    ...
	    NEXTOPARG();
	    goto *opcode_targets[opcode];
	}
	```

6. switch문을 사용할때:
	```C
	case LOAD_FAST: TARGET_LOAD_FAST: {
	    ...
	    NEXTOPARG();    
	    goto dispatch_opcode;
	}
	```

다음 그림은 switch문과 computed goto를 비교한 것이다:
![[cpython_ceval.png]]
### 예시
예시를 통해 Python 코드가 VM을 통해 어떻게 실행되는지 살펴보자.
```Python
def add(a, b):
    return a + b

add(2, 3)
dis.dis("add(2, 3)")    
```

```PlainText
0           RESUME                   0

1           LOAD_NAME                0 (add)
            PUSH_NULL
            LOAD_CONST               0 (2)
            LOAD_CONST               1 (3)
            CALL                     2
            RETURN_VALUE
```

1. 인터프리터에서 이 바이트코드를 실행하기 위해 스택 프레임으로 래핑된다. 그리고 평가를 위해 `_PyEval_EvalFrameDefault` 함수로 보내진다.
2. `_PyEval_EvalFrameDefault` 함수에서 디스패치 루프를 시작하기 전에, 인터프리터의 스택 프레임 최하단에 `entry_frame` 객체가 생성된다.
3. PVM의 명령어 포인터는 첫번째 명령어인 `LOAD_NAME`을 가리킨다.
4. 인터프리터가 디스패치 루프로 들어가고, 명령어 포인터가 순서대로 다음 명령어들을 가리키면서 명령어들이 하나씩 실행된다.
   모든 함수의 스택 프레임에는 고유한 명령어 포인터와 스택 포인터 값이 있다. 이 시점에서 호출자 프레임의 명령어 포인터는 add 함수의 실행이 완료되면 PVM이 반환할 RETURN_VALUE를 가리킨다.

5. `RETURN_VALUE` 명령어에 도달하면 스택 프레임이 pop되고 호출자에게 제어권이 반환된다.
6. 호출된 함수의 스택에서 반환 값을 가져온 다음 PVM의 스택 프레임을 pop하여 호출자의 스택 프레임이 다시 활성화되도록 한다.
7. PVM의 스택 포인터와 명령어 포인터가 각각 호출자 프레임, 즉 직전 프레임의 스택 포인터와 명령 포인터 값을 가리키도록 업데이트한다.
8. 직전 프레임의 다음 명령어를 실행한다.
9. 모든 명령어를 다 실행했다면, PVM의 스택 프레임에는 `entry_frame`만 남는다. `entry_frame`에는 `NOP`, `INTERPRETER_EXIT` 두 명령어만 존재한다.
    1. `NOP` 명령어는 아무 작업도 수행하지 않으므로 명령어 포인터는 다음 명령어로 이동한다.
    2. 다음 명령어인 `INTERPRETER_EXIT` 명령어는 디스패치 루프를 종료하고 인터프리터를 종료한다. 인터프리터 호출은 C 스택을 소모한다. 인터프리터를 종료할 때, 남은 C 스택 재귀 레벨을 업데이트할 수 있다.
### `INTERPRETER_EXIT` 명령어 동작
1. 명령어 포인터를 앞으로 이동시킨다.
2. entry_frame의 최상단 값을 가져와 반환 값으로 삼는다.
3. 스레드의 현재 프레임이 직전 프레임을 가리키도록 설정한다. 만약 직전 프레임이 없다면 NULL로 설정된다.
4. 인터프리터 호출은 C 스택을 소모하므로 남은 C 스택 재귀 레벨을 업데이트한다.
5. 인터프리터의 바이트코드 평가 루프에서 반환값을 반환하기 위해 `return`으로 끝난다. 이때 다음 명령어를 실행하기 위해 루프로 돌아가지 않는다.
6. Python 프로그램 실행이 종료되며, 실행된 코드 블록의 반환 값은 PVM이 호출된 위치로 반환된다.

> [!tip]
    > CALL 명령어는 함수 호출을 수행한다. 이를 위해 함수 호출자에 대한 새 스택 프레임을 생성하고 이를 활성 스택 프레임으로 만든다. 또한 호출자의 첫 번째 인스트럭션을 가리키도록 명령어 포인터 변수를 업데이트하고 마지막으로 디스패치 루프의 시작 부분으로 다시 점프하여 호출자의 코드를 실행한다.


# 3. CPython 3.13의 주요 특징 및 최적화
CPython 3.13 버전에서 관찰되는 주요 특징은 다음과 같다.
## Tiered Interpreter
-   **Tier 1 (일반 인터프리터)**: `_PyEval_EvalFrameDefault`가 바로 Tier 1 인터프리터의 구현체다. 모든 바이트코드를 실행할 수 있는 범용 실행 엔진이다.
-   **Tier 2 (최적화 인터프리터)**: 코드 실행 중 'hot' 코드 블록이 식별되면, 해당 코드는 더 빠르고 특화된 명령어(uops)로 구성된 **트레이스(trace)** 로 변환된다. 이 트레이스는 `_PyTier2Interpreter` (또는 JIT 컴파일러)에 의해 훨씬 빠르게 실행된다. Tier 1보다 훨씬 단순하고 예측 가능성이 높아 월등히 빠릅니다. 이는 반복적인 코드의 실행 속도를 크게 향상시키는 **전문화 적응형 인터프리터(Specializing Adaptive Interpreter)** 의 핵심이다.
## Inline Caching
속성(attribute) 로드, 전역 변수 접근 등 자주 발생하는 작업의 결과를 캐싱하여 다음 호출 시 더 빠르게 처리합니다. 바이트코드 명령어 자체를 실행 시점에 더 효율적인 버전으로 바꾸는 **적응형 옵코드(adaptive opcode)** 메커니즘을 통해 구현됩니다.

## 5. 결론
`ceval.c`는 Python 코드를 바이트코드 수준에서 실제로 생명을 불어넣는 CPython의 핵심 엔진입니다. 평가 루프, 프레임 관리, 값 스택 연산을 통해 프로그램의 흐름을 제어하며, CPython 3.13에서는 Tiered Interpreter와 같은 고급 최적화 기법을 도입하여 지속적으로 성능을 개선하고 있습니다.
