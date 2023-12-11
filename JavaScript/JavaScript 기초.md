> [!vscode에서 js파일 실행시키기]
> 1. coderunner 확장팩 설치
> 2. `ctrl + alt + n` 으로 실행

# 변수와 상수
- 변수: 바뀔수 있는 값, `let` 키워드 사용
- 상수: 바뀔수 없는 값, `const` 키워드 사용
- 다른 블록 범위에서는 변수명이 같아도 됨
- `null`: 값이 없음을 명시한 것
- `undefined`: 값을 할당하지 않아 값이 없는 상태
```js
const friend = null;
let criminal;

console.log(friend); //null 
console.log(criminal); //undefined
```

---
# 연산자
- 산술 연산자: `+`, `-`, `*`, `/`, `%`
	- `+`로 문자열 붙일 수 있음
- `a++`: 해당 코드를 실행한 후에 1 더함
- `++a`: 해당 코드를 실행하며 1 더함
- 대입 연산자: `+=`, `-=`, `*=`, `/=`
## 논리 연산자
- `!`: NOT
- `&&`: AND
- `||`: OR
- 연산 순서: NOT -> AND -> OR
## 비교 연산자
- `===`: 두 값과 타입이 모두 일치하는지 확인
- `!==`: 두 값과 타입이 모두 불일치하는지 확인
- `==`: 두 값이 일치하는지만 확인
- `!=`: 두 값이 불일치하는지만 확인
- `<`, `>`, `<=`, `>=`

---
# 조건문
- `if`, `if else`, `else`
- switch/case문은 자바와 똑같음

---
# 템플릿 리터럴
- `+`를 사용하지 않고 문자열을 묶는 방법
- **작은 따옴표가 아니라 역따옴표임을 주의!!**
```js
function hello(name) {
    console.log(`hello, ${name}`);
}

hello('siwoli');

let number = 10;
console.log(`number: ${number}`);
```

---
# 화살표 함수
- 화살표 좌측에는 함수의 매개변수
- 화살표 우측에는 코드 블록
- 무조건 익명함수로만 사용 가능
- 메서드나 생성자 함수로 사용 불가
- 코드 블록 내부에서 바로 return하는 경우에는 블록과 return 생략 가능
```js
const diff = (a, b) => a-b;
console.log(diff(3,2));
```

**화살표 함수의 this 바인딩**
- 화살표 함수에는 this라는 변수 자체가 존재하지 않기 때문에 그 상위 환경에서의 this 참조
[화살표 함수 this](https://velog.io/@padoling/JavaScript-%ED%99%94%EC%82%B4%ED%91%9C-%ED%95%A8%EC%88%98%EC%99%80-this-%EB%B0%94%EC%9D%B8%EB%94%A9)
