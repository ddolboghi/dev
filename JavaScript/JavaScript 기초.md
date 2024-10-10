> [!vscode에서 js파일 실행시키기]
> 1. coderunner 확장팩 설치
> 2. `ctrl + alt + n` 으로 실행

# `parseInt(string, radix)`
- _@param_ `string` -- 숫자로 변환할 문자열
- _@param_ `radix` -- 선택사항, string 문자열을 읽을 진법, 2~36의 수
- _@return_ `int` -- string을 정수로 변환한 값 리턴, string의 첫 글자를 정수로 변경할 수 없으면 NaN(Not a Number)리턴

# `.toFixed(digits: number)`
- 주어진 digits만큼의 소수점 이하 자리수를 정확하게 갖는 **문자열로 반환**
- 소수점 이하가 길면 반올림하고, 짧으면 0으로 채움
# 변수와 상수
- 변수: 바뀔수 있는 값, `let` 키워드 사용
- 상수: 바뀔수 없는 값, `const` 키워드 사용
- `let`과 `const`는 블록레벨의 스코프를 가지기때문에 다른 블록 범위에서는 변수명이 같아도 됨
- `var`은 함수에서만 지역변수가 되는 함수레벨 스코프를 가짐
- `null`: 값이 없음을 명시한 것
- `undefined`: 값을 할당하지 않아 값이 없는 상태
```js
const friend = null;
let criminal;

console.log(friend); //null 
console.log(criminal); //undefined
```

> [!tip]
> Object는 let으로 선언하는게 좋다. 왜냐하면 const로 선언해도 Object안의 property 값들은 수정할 수 있기 때문이다.
# scope
- 전역 스코프: 코드의 모든 범위에서 사용 가능
- 함수 스코프: 함수 안에서만 사용 가능. `var`
- 블록 스코프: if, for, switch등 특정 블록 내부에서만 사용 가능. `let`, `const`

# hoisting
- 자바스크립트 엔진이 코드에서 모든 선언문을 찾아 먼저 실행해 필요한 메모리 공간을 미리 할당하는 것
- `var`, `function`, `function*`, `class`로 선언한 모든 식별자는 호이스팅되어 먼저 선언됨
- `var`은 호이스팅되어 undefined로 변수를 초기화해줌
- `let`, `const`는 호이스팅되지만 변수 생성과정이 달라 일시적인 사각지대가 생성되어 초기화전에 엑세스할 수 없음
- 호이스팅은 방지하는게 유지보수에 좋음
	- 함수는 반드시 선언 후 호출하기
	- `var`대신 `let`, `const`사용하기

## 함수 정의 방식에 따른 호이스팅
```js
// 1. 함수 선언문
function add(a, b) {
  return a + b
}

// 2. 함수 표현식 (함수 이름 생략 가능)
const add = function (a, b) {
  return a + b
}

// 3. 화살표 함수 (리턴값만 있는 경우 한줄로 표현 가능)
const add = (a, b) => a + b

// 4. Function 생성자 함수
const add = new Function('a', 'b', 'return a + b')
```
- 함수 선언문은 호이스팅 적용됨
- 함수 표현식(2,3,4)은 변수에 담겨 변수의 호이스팅 규칙을 따름

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
- `===`: 두 값과 타입이 모두 일치하는지 확인, **타입 강제변환을 허용하지 않음**
- `!==`: 두 값과 타입이 모두 불일치하는지 확인, **타입 강제변환을 허용하지 않음**
- `==`: 두 값의 타입이 값다면 일치하는지만 확인(강제 변환 동등 비교 연산자), **타입이 다르면 타입 강제변환 후 비교한다. 보통 숫자형으로 변환한다.**
- `!=`: 두 값이 불일치하는지만 확인, **타입 강제변환**을 허용함
- `<`, `>`, `<=`, `>=` : **타입 강제변환**을 허용함, 보통 숫자형으로 변환하고 값을 비교함
> [!tip]
> - `Object.is(value1, value2) : boolean`를 사용하면 매우 정확하게 일치하는지 판단해준다.
> - `NaN`과 비교할땐 `Number.isNaN(value)`를 사용해라
> - js는객체의 일치 여부(`===`)를 판단할때 구조적 일치가 아닌 독자성 일치를 비교한다. 
>   따라서 객체의 참조를 비교하기 때문에 같은 구조라도 다른 참조의 객체들은 `===`비교 시 불일치한다.
> - js에는 구조적 일치를 비교할 방법이 없다. 직접 코드를 작성해야한다.
> - 타입 강제변환을 허용하는 비교 연산자에서 모든 피연산자가 문자열이면 숫자형으로 변환하지 않고 알파벳순으로 문자를 비교한다.

# 조건문
- `if`, `if else`, `else`
- switch/case문은 자바와 똑같음

# 템플릿 리터럴
- `+`를 사용하지 않고 문자열을 묶는 방법
- **작은 따옴표가 아니라 역따옴표(backtic)임을 주의!!**
```js
function hello(name) {
    console.log(`hello, ${name}`);
}

hello('siwoli');

let number = 10;
console.log(`number: ${number}`);
```

# 화살표 함수
- 화살표 좌측에는 함수의 매개변수
- 화살표 우측에는 코드 블록`{}`
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
- 일반 함수를 메서드로 호출해야 자신을 호출한 객체인 `cat`을 가리킴

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

# 객체
```javascript
const dog = {
    name: '시월이',
    age: 2,
    '습관': '뛰기'
};

console.log(dog);
console.log(dog.습관);
```
- `객체.entries`: `[[키, 값], [키, 값], [키, 값]]`형태의 배열로 변환
- `객체.keys`: 키들만 모아서 배열로 변환
- `객체.values`: 값들만 모아서 배열로 변환
## 함수에서 객체를 파라미터로 받기
### 템플릿 리터럴 이용하는 방법
```javascript
function print(dog) {
    const text = `${dog.name}, ${dog.age}살`
    console.log(text);
}
```
### 객체 비구조화 할당(=객체 구조 분해)
```javascript
function print(dog) {
    const {name, age} = dog; //객체에서 값들을 추출해 새로운 상수로 선언
    const text = `${name}, ${age}살`
    console.log(text);
}
```

```js
function print({name, age}) {
    const text = `${name}, ${age}살`
    console.log(text);
}
```
## 객체 안에 있는 함수
- 객체 안의 함수를 호출할때는 `객체.함수 정의한 속성명()`으로 괄호 반드시 붙여야함
- 화살표 함수를 사용하면 this는 함수를 호출한 객체가 아닌 전역 객체를 가리키므로 일반함수를 사용해야 속한 객체를 가리킬 수 있음
```js
const dog = {
    name: '시월이',
    age: 2,
    say: function() {
        console.log(this.name);
    }
};

dog.say();
```

```js
var testObj = {
    myName: "jjy",
    greeting(name) {
        console.log(`thank you, ${name} ${testObj.myName}`); //this.myName을 사용해도 된다.
    }
};
testObj.greeting("jjy")
// thank you, jjy jjy
```
## Getter & Setter
```js
const dog = {
    name: '시월이',
    age: 2,
    say: function() {
        console.log(this.name);
    },
    get getAge() {
        return this.age;
    },
    set setAge(age) {
        this.age = age;
    }
};

console.log(dog.getAge); // 2
dog.setAge = 10;
console.log(dog.getAge); // 10
```
- Getter: `get`키워드 사용, js가 함수로 취급하지 않아 속성처럼 사용해야함
- Setter: `set`키워드 사용, Getter와 마찬가지로 속성처럼 사용하며 값 지정
## 객체 생성자
- 함수를 통해 새로운 객체를 만들고 그 안에 넣고 싶은 값이나 함수들을 구현
- 객체 생성 함수 이름은 **대문자**로 시작
- `new`키워드로 객체 생성(자바랑 유사)
```js
function Animal(type, name, sound) {
    this.type = type;
    this.name = name;
    this.sound = sound;
    this.say = function() {
        console.log(this.sound);
    };
}

const dog = new Animal('시바견', '시월이', '멍멍');

dog.say();
```

# 프로토타입
- 같은 객체 생성자 함수를 사용하는 경우, 특정 함수 또는 값을 재사용
- `객체생성자함수명.protorype.[원하는 키] = 함수 또는 값`
- ES6에서 class 문법이 추가되기 전에 class를 구현하기 위해 사용함

# 클래스
```js
class Animal {
    constructor(type, name, sound) {
        this.type = type;
        this.name = name;
        this.sound = sound;
    }
    say() {
        console.log(this.sound);
    }
}

const dog = new Animal('시바견', '시월이', '멍ㅁ어');

dog.say();
```
- 메서드: 클래스 내부에 선언한 함수 e.g. say()
- 메서드는 자동으로 프로토타입으로 등록됨
## 상속
- `하위클래스 extends 상위클래스`
- 하위클래스의 `constructor()`에는 하위클래스에서 사용할 속성을 지정
- `super()`로 상위 클래스의 생성자를 모두 가져와야함
- `super()`안에 `constructor()`에 지정한 속성이 있어야함
- 상위클래스의 일부 속성만 하위클래스에서 지정하려면 `super()`로 가져온 일부 속성에 값을 지정하면 상위 클래스의 속성 값이 아닌 지정된 값으로 가져오게됨
```js
class Dog extends Animal {
    constructor(name, sound) {
        super('개', name, sound);
    }
}

const dog2 = new Dog('백기', '왈');

dog2.say();
```

# 배열
- `배열.length`: 배열의 길이
- `배열[index]`: index에 해당하는 배열의 원소
- `배열.push(값)`: 배열에 값 추가
- `배열.at(-index)`: 파이썬 처럼 음수 인덱싱 가능
```js
const arr = [1,2,3,'4'];
console.log(arr[0]); // 1
console.log(arr[arr.length-1]); // 4

const objectArr = [{name:'a'}, {name:'b'}];
console.log(objectArr[0]); // {name: 'a'}
```
## 배열 내장 함수
- 파라미터로 전달된 콜백 함수는 동기식 콜백(synchronous callbacks)임

| 함수                             | 설명                                                                                                                                                                                                                                                |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.forEach(콜백 함수)`              | 배열의 각 원소 반복                                                                                                                                                                                                                                       |
| `.map(콜백 함수)`                  | - 배열 안의 각 원소를 변환할때 사용<br> - 파라미터로 원소에 변화를 주는 함수를 사용해야함                                                                                                                                                                                            |
| `.indexOf(원소)`                 | - 해당 원소의 인덱스 반환<br> - 객체나 배열이 아닌 원소만 가능                                                                                                                                                                                                           |
| `.findIndex(콜백 함수)`            | - 조건을 반환하는 콜백 함수를 넘겨줘야함<br>- 조건에 맞는 원소의 인덱스 반환<br>- 객체나 배열이 원소일때도 활용 가능                                                                                                                                                                           |
| `.find(콜백 함수)`                 | - 조건을 반환하는 콜백 함수를 넘겨줘야함<br>- 조건에 맞는 원소 자체를 반환                                                                                                                                                                                                     |
| `.filter(콜백 함수)`               | - 조건을 반환하는 콜백 함수를 넘겨줘야함<br>- 조건에 **true**인 원소들을 추출해 새로운 배열을 만들어 반환                                                                                                                                                                                |
| `.splice(startIndex, 지울 개수)`   | - 배열에서 특정 항목 제거<br>- 원본 배열이 변경됨                                                                                                                                                                                                                   |
| `.slice(startIndex, endIndex)` | - 배열을 자르고 반환<br>- endIndex-1번째까지 자름<br>- 원본 배열을 건드리지 않고 새로운 배열 생성됨<br>- 아무 값도 없으면 처음부터 끝까지 복사된 배열 생성됨                                                                                                                                             |
| `.shift()`                     | 원본 배열의 첫번째 원소 삭제하고 반환                                                                                                                                                                                                                             |
| `.pop()`                       | 원본 배열의 마지막 원소 삭제하고 반환                                                                                                                                                                                                                             |
| `.unshift(원소)`                 | 원본 배열의 맨 앞에 새 원소 추가                                                                                                                                                                                                                               |
| `.concat(배열)`                  | - 여러 개의 배열을 하나의 배열로 합친 새로운 배열 반환<br>- 원본 배열에 영향X                                                                                                                                                                                                  |
| `.join(delimiter)`             | - 배열의 원소들을 문자열 형태로 합침<br>- delimiter 지정 안하면 `,`로 구분됨                                                                                                                                                                                              |
| `.reduce(콜백 함수, 초기값)`          | 콜백 함수에서 사용하는 파라미터:<br>- `accmulator`: 콜백 함수의 반환값 누적, 초기값을 지정한 경우 콜백 함수의 첫 번째 호출 값은 초기값임<br>- `current`: 현재 처리하고 있는 원소<br>- `currentIndex`: 현재 처리하고 있는 원소의 인덱스, 초기값을 지정한 경우 0 아니면 1 <br>- `array`: 현재 처리하고 있는 배열 자신<br>초기값은 `reduce`함수에서 사용하는 초기값임 |
| `.includes(원소)`                | 배열에 해당 원소가 포함돼있는지 true/false 반환                                                                                                                                                                                                                   |

- `.reduce()`는 배열뿐만 아니라 iterator객체에 적용 가능! (e.g. rest객체 등)
```js
const arr = [1,2,3,4,5];
 arr.forEach(e => {
     console.log(10*e);
 })
```

```js
const arr = [1,2,3,4,5];
const square = arr.map(e => e * e);
console.log(square);
```

```js
let mean = arr.reduce((acccumulator, current, index, array) => {
    if (index === array.length-1) {
        return (acccumulator+current) / array.length;
    }
    return acccumulator + current;
}, 0);

console.log(mean);
```

# 반복문
- `continue`: 다음 루프 실행
- `break`: 반복 종료
## for - of 문
- 자바의 for - each와 유사
- 보통 배열을 반복할때는 for -of보다 배열 내장함수를 많이 사용함
```js
const arr = [1,2,3,4,5];
for (let e of arr) {
    console.log(e);
}
```
## for - in 문
- 객체의 key로 객체가 지닌 속성 돌때 사용
```js
const dog = {
    name: '시월이',
    age: 2,
    length: 70
}

for (let key in dog) {
    console.log(key, dog[key]);
}
```

# 콜백 함수
- 전달 인자로 다른 함수에 전달되는 함수
- 동기식 콜백(synchronous callbacks): 중간에 비동기 작업 없이 외부 함수 호출 직후에 호출
- 비동기식 콜백(asynchronous callbacks): 동기 작업이 완료된 후 나중에 호출됨
[참고](https://inpa.tistory.com/entry/JS-%F0%9F%93%9A-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%BD%9C%EB%B0%B1-%ED%95%A8%EC%88%98)

# ES2022
[참고](https://fe-developers.kakaoent.com/2022/220728-es2022/)

# 클로저
- 함수가 정의된 스코프가 아닌 다른 스코프에서 함수가 실행되더라도, 스코프 밖에 있는 변수를 기억하고 이 외부 변수에 계속 접근할 수 있는 경우를 의미한다.
- 클로저는 함수의 타고난 특징이다.
- 객체는 클로저가 되지 않지만, 함수는 클로저가 된다.
- 클로저를 직접 보려면 함수를 해당 함수가 정의된 스코프가 아닌 다른 스코프에서 실행해야 한다.
```js
function greeting(msg) {
	return function who(name) {
		console.log(`${name}, ${msg}!`);
	};
}

var hello = greeting("hello!");
var howdy = greeting("how are you?");
hello("bora"); // bora, hello!
howdy("naen"); // naen, how are you?
```
1. `greeting()`이 실행되고 `who()`