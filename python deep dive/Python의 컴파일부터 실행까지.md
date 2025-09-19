CPython은 소스 코드를 구문분석하는 파서와, 바이트코드로 만드는 컴파일러, 그 바이트코드를 실행하는 PVM(인터프리터) 등을 모두 포함하는 하나의 거대한 시스템이다. 이 시스템을 큰 범주에서 Python Interpreter라고 부르기도 한다.

```Mermaid
graph TD
    subgraph CPython 컴파일러
        A["source.py (소스 코드)"] --> B{"1. 어휘 분석 (Lexing)"};
        B --> C["토큰 (Tokens)"];
        C --> D{"2. 구문 분석 (Parsing)"};
        D --> E["구문 트리 (Parse Tree)"];
        E --> F{"3. AST 생성"};
        F --> G["추상 구문 트리 (AST)"];
        G --> H{"4. 컴파일 (Compile)"};
        H --> I["bytecode 생성"];
        I --> J["bytecode 최적화"];                
    end

    subgraph PVM 인터프리터
        K["source.pyc (바이트코드)"] --> L{"5. 실행 (Evaluation Loop)"};
        L --> M["프로그램 결과"];
    end

    J--> K;

```
컴파일러는 .py 소스 코드를 읽어 어휘 분석, 구문 분석, AST(추상 구문 트리) 생성을 거쳐 PVM이 이해할 수 있는 바이트코드를 생성한다. compile.c에 구현되있다. .py 파일의 Python 코드가 컴파일된 바이트코드는 __pycache__ 디렉터리 안에 .pyc 파일로 저장된다.

**PVM은 스택 기반의 인터프리터**다. PVM의 evaluation loop는 컴파일된 바이트코드를 한 줄씩 읽어들이고, 각 명령어(co_code)에 해당하는 C함수를 실행하여 프로그램을 구동시킨다. 바이트코드를 한 줄씩 실행하기 때문에 pipelining, optimazation이 줄 별로만 가능하다.

evaluation loop, 즉 바이트코드 인터프리터는 ceval.c에 구현되있는 거대한 switch문이다. switch문의 각 case는 바이트코드 명령어이며, 이 명령어로 어떤 동작을 수행하는지 적혀있다.

# Evaluation loop 동작
[[Evaluation loop]]

### Pythonic 코드와 바이트코드 최적화
코드를 Pythonic하게 작성하면 컴파일러는 전용 바이트코드 명령어로 컴파일하기 때문에 더 빠르고 효율적인 바이트코드를 생성하여 PVM의 실행 부담을 크게 줄일 수 있다.

#### List Comprehension
```Python
squares = []
for i in range(10):
    squares.append(i * i)
```
이 코드는 다음과 같은 바이트코드(개념적으로) 생성한다.
1. 빈 리스트 squares를 만든다.
2. 루프를 시작한다.
3. 매 반복마다:
    1. squares 변수를 찾는다.
    2. .append라는 속성(메서드)를 찾는다.
    3. i * i를 계산한다.
    4. 찾아낸 append 메서드를 호출한다.
4. 루프를 끝낸다.

매번 반복할때마다 squares와 .append를 다시 찾아야 하므로 PVM 입장에서 비효율적인 작업이다.
 
리스트 컴프리핸션 사용 시:
```Python
squares = [i * i for i in range(10)]
```
1. 내부적으로 빈 리스트를 만든다.
2. 루프를 시작한다.
3. 매 반복마다:
    1. i * i를 계산한다.
    2. 계산된 값을 LIST_APPEND라는 특수 명령어로 리스트에 바로 추가한다.
4. 루프를 끝내고 완성된 리스트를  반환한다.

LIST_APPEND라는 전용 바이트코드 명령어는 C언어 수준에서 매우 빠르게 동작하도록 구현되어 있다. 변수나 속성을 일일이 다시 찾는 과정이 완전히 생략되므로 훨씬 효율적이다.

#### Constant Folding
코드에 직접 쓰인 상수 계산은 컴파일 단계에서 미리 계산하여 결과값을 바이트코드로 변환한다. 이 때문에 런타임에 계산할 필요가 없어 더 빠르다.

상수를 변수에 할당했을때:
```Python
def slow_week():
  seconds_per_day = 86400
  return 7 * seconds_per_day
```

위 함수가 바이트코드로 변환되면 다음과 같다:
```PlainText
0 LOAD_CONST    1 (86400)
2 STORE_FAST    0 (seconds_per_day)

4 LOAD_CONST    2 (7)
6 LOAD_FAST     0 (seconds_per_day)
8 BINARY_MULTIPLY
10 RETURN_VALUE
```
6개의 바이트코드 명령어가 있다.

상수를 코드에 직접 사용했을때:
```Python
def fast_week():
  return 7 * 86400
```

위 함수가 바이트코드로 변환되면 다음과 같다:
```PlainText
0 LOAD_CONST    1 (604800)
2 RETURN_VALUE
```
변수를 저장하지 않아 2개의 바이트코드 명령어로 줄었다.
이렇게 1.5배 성능 향상을 이룰 수 있다.

#### Small Integer Caching
CPython은 -5부터 256까지의 정수를 미리 만들어놓고 재사용한다(캐싱). 그래서 a=100, b=100일때 a is b는 True지만, a=1000, b=1000일때 a is b는 False가 된다.

참고로 None을 비교할때는 is를 사용하는 것이 관례다.

참고: [https://aosabook.org/en/500L/a-python-interpreter-written-in-python.html](https://aosabook.org/en/500L/a-python-interpreter-written-in-python.html), [https://youtu.be/tzYhv61piNY?si=Ip2Ps2yZ2ENfu8Wy](https://youtu.be/tzYhv61piNY?si=Ip2Ps2yZ2ENfu8Wy)