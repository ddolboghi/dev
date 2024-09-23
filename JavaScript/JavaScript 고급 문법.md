# 삼항 연산자
`조건 ? true일때 : false일때`

# Truthy and Falsy
- Truthy: ture처럼 여기는 값. falsy한 값을 제외한 모든 값
- Falsy: false처럼 여기는 값. undefined, null, 0, ' ', NaN
- 어떤 값이 truthy한 값이면 true, 아니면 false로 표현하는 방법
```js
const value = {a:0};
const truthy = !!value;
console.log(truthy); // false
```

# 단축 평가 논리 계산법
## A && B 
- A가 truthy한 값이면 결과는 B가 되고, A가 falsy한 값이면 결과는 A가 됨
- 특정 값이 유효할때만 어떤 값을 조회해야할때 유용함
```js
console.log(true && 'hello'); // hello
console.log(false && 'hello'); // false
console.log('hello' && 'bye'); // bye
console.log(null && 'hello'); // null
console.log(undefined && 'hello'); // undefined
console.log('' && 'hello'); // ''
console.log(0 && 'hello'); // 0
console.log(1 && 'hello'); // hello
console.log(1 && 1); // 1
```
## A || B
- A가 truthy한 값이면 결과는 A가 되고, A가 falsy한 값이면 결과는 B가 됨
- 어떤 값이 falsy하다면 대신 사용할 값을 지정해줄때 유용함
```js
const namelessDog = {
    name: ''
}

function getName(animal) {
    const name = animal && animal.name;
    return name || 'no name';
}

const name = getName(namelessDog);
console.log(name);
```
## A ?? B
- A가 null, undefined면 B 리턴하고, 그렇지 않으면 A 리턴
- falsy 값은 코드에서 의미있는 값일때 `||`연산자 대신 사용
- 우선순위: `||` > `??` > `? :`
```js
const foo = 'adsf' || 'default string';
console.log(foo); // adsf

const baz = null ?? 42;
console.log(baz); // 42
```

# 함수의 파라미터 기본값 지정
- 파라미터의 기본 값 지정
- 화살표 함수에서도 사용할 수 있음
```js
function calculateCircleArea(r = 1) {

    return Math.PI * r * r;

}
console.log(calculateCircleArea());

const calculateSqureArea = (d = 1) => d*d;
console.log(calculateSqureArea())
```

# 특정 값이 여러 값 중 하나인지 확인할때
- 배열 내장함수 `includes()`활용하기
```js
const isAnimal = name => ['고양이', '개', '거북이', '너구리'].includes(name);

console.log(isAnimal('개')); // true
console.log(isAnimal('노트북')); // false
```

# 값에 따라 다른 결과물을 반환해야 할때
- 객체 활용
```js
function getSound(animal) {
    const sounds = {
        '개':'www'
    };
    return sounds[animal] || '...';
}

console.log(getSound('개')); // www
console.log(getSound()); // ...
```
- 값에 따라 다른 코드를 실행해야할때는 객체에 함수 넣기
```js
function makeSound(animal) {
    const tasks = {
        dog() {
            console.log('멍멍');
        },
        cat() {
            console.log('야옹');
        }
    };
    if (!tasks[animal]) {
        console.log('...');
        return;
    }
    tasks[animal]();
}

makeSound('dog');
```

# 비구조화 할당(구조분해) 문법
- 객체 안에 있는 값을 추출해서 변수나 상수로 바로 선언할 수 있음
- 비구조화 할당은 얕은 복사이므로 비구조화 할당후 값을 변경해도 원본값은 변하지 않음
## 비구조화 할당시 기본값 설정
- 객체에 없는 속성을 사용하려면 기본값 지정해야함. 안그러면 undefined
```js
const ob = {a:1}
const {a, b=2} = ob;

console.log(a); // 1
console.log(b); // 2

function print({a, b=2}) {
    console.log(a);
    console.log(b);
}

print(ob);
// 1
// 2
```
## 비구조화 할당시 이름 바꾸기
- 객체 속성명과 변수명이 다를때 `속성명: 변수명`으로 속성명 바꿀수 있음
```js
const animal = {
    name: 'siwoli',
    type: 'dog'
};

const {name: nickname} = animal
console.log(nickname);
```
## 배열 비구조화 할당
- 배열 안에 있는 원소를 다른 이름으로 새로 선언해주고 싶을때 사용
- 없는 원소의 기본값 지정도 가능
```js
const arr = [1,2,3,4,5]
const [a,b,c,d,e] = arr;

console.log(a); // 1
console.log(e); // 5
```
## 깊은 값 비구조화 할당
- 객체 안에 객체가 있는 중첩된 구조에서 모든 속성값들을 추출할때 객체 구조 그대로 비구조화 할당하면 됨
```js
const deepOb = {
    address: {
        nation: {
            name: 'Korea',
            nationNumber: 82
        },
        city: 'Busan'
    },
    language: 'korean'
};
  
const {
    address: {
        nation: {name, nationNumber},
        city
    },
    language
} = deepOb;

const extracted = {
    name, nationNumber, language
};

console.log(extracted);
```

# object-shorthand 문법
- 객체를 선언할때, 객체의 key와 같은 이름으로 선언된 값이 존재하면 바로 매칭시켜줌
```js
let name = 'aaa';
let age = 10;

const info = {
    name: name,
    age: age
}

console.log(info.name);
console.log(info.age);
```

# spread
- `...`연산자를 사용해 **iterator**를 펼침
- `...`는 iterator를 소비하는 주체인 iterator consumer로, `for ...of`반복문과 마찬가지로 iterator 소비 프로토콜을 준수함
- 배열에서 여러 번 사용가능
- 배열안에 사용 시 iterator를 펼친 결과가 배열의 각 요소가 되도록 함
```js
const arr = [1,2];
const brr = [...arr, 3, ...arr];

console.log(brr); // [1,2,3,1,2]
```

```js
function sum(...rest) {
    return rest.reduce((acc, current) => acc + current, 0);
}

const numbers = [1,2,3,4,5,6];

const result = sum(...numbers);
console.log(result);
```

# rest
- `...변수명`으로 사용
- 객체, 배열, 함수의 파라미터에서 사용 가능
- 객체와 배열에서 사용할때는 비구조화 할당 문법과 함께 사용됨
	- rest 연산자는 항상 마지막에 와야함
	- 비구조화 할당하지 않은 나머지 속성(원소)을 rest가 가짐
```js
const numbers = [1,2,3,4,5];
const [one, ...rest] = numbers;
console.log(rest); // [2,3,4,5]
```

```js
const ob = {
    a:1,
    b:2,
    c:3
};

const {a, ...e} = ob;
console.log(a); // 1
console.log(e); // { b: 2, c: 3 }
```

- 함수의 파라미미터가 몇 개일지 모를때 rest사용하면 유용함
```js
function sum(...rest) {
    return rest.reduce((acc, current) => acc + current, 0);
}

const result = sum(1,2,3,4,5,6);

console.log(result);
```

- rest와 reduce로 최대값 구하기
```js
function max(...numbers) {
	return numbers.reduce((acc, current) => (current > acc ? current : acc), numbers[0]);
}

const result = max(1,2,3,4,10,5,6,7);
console.log(result);
```
# 옵셔널 체이닝
- `?.`은 `?.`'앞’의 평가 대상이 `undefined`나 `null`이면 평가를 멈추고 `undefined`를 반환합니다.
```js
let user = {}; // 주소 정보가 없는 사용자
alert(user?.address?.street); // undefined, 에러가 발생하지 않습니다.
```
- `?.`앞의 변수는 꼭 선언되어 있어야 합니다.
- `?.`는 평가대상에 값이 없으면 즉시 평가를 멈추는 단락 평가를 합니다.
- 함수 호출을 비롯한 `?.` 오른쪽에 있는 부가 동작은 `?.`의 평가가 멈췄을 때 더는 일어나지 않습니다.