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
- 클래스의 메서드나 생성자로 사용 불가
- 코드 블록 내부에서 바로 return하는 경우에는 블록과 return 생략 가능
```js
const diff = (a, b) => a-b;
console.log(diff(3,2));
```

**화살표 함수의 this 바인딩**
- 화살표 함수에는 this라는 변수 자체가 존재하지 않기 때문에 화살표 함수가 선언될 시점의 상위 스코프의 this로 바인딩됨

```js
const cat = {
  name: 'meow';
  callName: () => console.log(this.name);
}

cat.callName();	// undefined
```
- 이 경우, `callName` 메소드의 `this`는 자신을 호출한 객체 `cat`이 아니라 함수 선언 시점의 상위 스코프인 전역객체를 가리킴
- 일반 함수를 메서드로 호출해도 자신을 호출한 객체를 가리키므로 일반 함수를 사용해도 됨

```js
const button = document.getElementById('myButton');

button.addEventListener('click', () => {
  console.log(this);	// Window
  this.innerHTML = 'clicked';
});

button.addEventListener('click', function() {
   console.log(this);	// button 엘리먼트
   this.innerHTML = 'clicked';
});
```
- `addEventListener`의 콜백함수는 this에 해당 이벤트리스너가 호출된 엘리먼트가 바인딩되도록 정의되있음. 즉, this가 button을 가리킴
- `addEventListener`안에 화살표 함수를 사용하면 기존 this의 바인딩 값이 사라지고 상위 스코프가 바인딩됨

일반 함수의 this 바인딩
- 함수를 호출했을때 그 함수 내부의 this는 지정되지 않음
- this가 지정되지 않은 경우,  this는 자동으로 전역 객체를 가리킴
- 함수 내부의 this는 전역 객체가 됨

---
# 객체