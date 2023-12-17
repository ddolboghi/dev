# React 특징
- 리액트 자체는 라이브러리
- 리액트 기반의 프레임워크: Next.js, Remix, Gatsby 등
## 컴포넌트 기반
- React 코드는 컴포넌트라는 개체로 구성
- 컴포넌트는 JSX 문법을 사용해 마크업을 반환하는 자바스크립트 클래스나 함수로 이뤄져있으며, 속성(props)을 입력으로 받고 UI의 어느 부분이 나타나야 하는지를 설명하는 React 요소 반환
- 컴포넌트 내에서 이벤트 핸들러 함수 정의
- 컴포넌트는 썸네일, 좋아요 버튼, 비디오 등 개별적인 조각 또는 전체 페이지처럼 큰 조각들을 통칭함
- 컴포넌트를 생성하고 이들을 결합해 사용자 인터페이스 구성
- 여러 컴포넌트가 동일한 데이터를 공유하고 조작해야할때는 상태를 가장 가까운 공통 조상으로 끌어올린 다음 이 상태를 자식 컴포넌트에 props로 전달
## 선언적
- React는 뷰가 데이터 변경에 자동으로 맞춰지게 함
- 데이터가 변경될때 React가 적절한 컴포넌트를 업데이트하고 렌더링함
- 코드를 더 가독성있게 만들어 디버깅이 쉬워짐
## 가상 DOM
- React는 실제 DOM의 가벼운 복사본인 가상 DOM을 생성함
- 조정(reconciliation): 객체의 상태가 변경되면 React는 가상 DOM에 작업 후 이전 가상 DOM과 새로운 가상 DOM의 차이 비교해 실제 DOM에서 실제로 변경된 부분만 업데이트
## 단방향 데이터 바인딩과 FLUX
- React는 단방향 데이터 흐름을 구현해 애플리케이션을 이해하기 쉽게 만듬
- FLUX는 데이터의 단방향 유지를 돕는 패턴
- React 앱 설계시 종종 부모 컴포넌트 내에 자식 컴포넌트를 중첩시켜 FLUX 구현
## 라우팅과 전역 상태 관리
- React 16.8 이후로 Hooks 기능을 통해 함수형 컴포넌트에서도 상태 관리 가능
- Redux 같은 추가적인 라이브러리로도 상태관리 가능
- React-Router로 라우팅 기능 사용
## JSX
- 자바스크립트 확장 마크업
- HTML보다 엄격하며 모든 태그를 닫아야함
- 스타일링을 위해 `className`속성을 사용해 CSS 클래스 지정 가능
- 중괄호를 사용해 자바스크립트 표현식(변수 등) 삽입 가능
  --> 자바스크립트 코드의 변수를 기반으로 동적으로 데이터 표시
- JSX를 관련 렌더링 로직 근처에 배치하면 컴포넌트를 쉽게 생성, 유지, 삭제할 수 있음
## 필요한 곳마다 상호작용성 추가
- 컴포넌트는 데이터를 받아들이고 화면에 표시해야할 내용 반환
- 사용자가 입력란에 입력하는 등의 상호작용에 대해 새로운 데이터를 컴포넌트에 전달하면 React는 화면을 새로운 데이터와 일치하도록 업데이트함
- 페이지 전체를 React로 구축할 필요 없이 기존의 HTML페이지에 React를 추가하고 인터랙티브한 컴포넌트를 어디서든 렌더링할 수 있음
---
# React 프로젝트 생성하기
1. 터미널에 `npx create-react-app [프로젝트 폴더명]`입력
	- [[설치 관련]](프로젝트 생성시 오류 참고)
2. `npm start`입력후 http://localhost:3000/ 열리는지 확인

---
# 컴포넌트 생성 및 중첩
```JavaScript
import React from 'react';

function Hello() {
    return <div>안녕하세요</div>
}
export default Hello;
```
- **컴포넌트 이름은 항상 ==대문자==로 시작**
- HTML 태그는 항상 소문자로 시작
- `export default`: 모듈에 별명을 부여해 해당 모듈을 다른 코드에서 가져오고 사용하기 쉽게 만듬
- 명명된 내보내기(named export)는 모듈에서 가져올 내용을 정확히 지정해야함
- `export default`와 명명된 내보내기 둘다 동일한 모듈에서 사용 가능
- 컴포넌트는 여러 개의 JSX 태그를 반환할 수 없음
	- 여러 개의 JSX 태그를 공유 부모(`<div>...</div>나 <>...</>`)로 묶어야함
	- JSX로 변환할 HTML이 많다면 [온라인 변환 도구](https://transform.tools/html-to-jsx)사용
```js
function AboutPage() {
  return (
    <>
      <h1>About</h1>
      <p>Hello there.<br />How do you do?</p>
    </>
  );
}
```

---
# 스타일 추가
- 컴포넌트 파일의 상단에 `import [css파일 경로]`추가
- html태그 안에 `className`속성으로 css 클래스 지정
```js
import React from 'react';
import './MyComponent.css';  // CSS를 import합니다.

function MyComponent() {
  return (
    <div className="container">
      <h1>Hello, World!</h1>
    </div>
  );
}

export default MyComponent;
```

---
# 데이터 표시
- JSX에서 태그 사이 **중괄호** 안에 JS의 변수를 넣어 데이터 표시
- JSX 속성에도 중괄호로 데이터 표시 가능
```JS
return (
  <h1>
    {user.name}
  </h1>
);
```
```JS
return (
  <img
    className="avatar"
    src={user.imageUrl}
  />
);
```

---
# 조건부 렌더링
- 특별한 구문 없이 자바스크립트의 조건문(`if`문) 활용
- 속성을 조건부로 지정하는것도 조건문 활용하면 됨

---
# 목록 렌더링
- `for`문 또는 `배열.map()`함수 사용
```js
const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```
- `<li>`의 key속성에 목록의 각 항목을 고유하게 식별하는 값 전달해야함(일반적으로 데이터베이스 id값)

---
# 이벤트 처리

```js
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```
- JSX에서 이벤트핸들러 함수를 호출하지 않고 **html 태그의 이벤트에 함수를 전달함**(이때 함수 끝에 `()`안 붙임)
- 리액트 이벤트는 소문자 대신 카멜케이스로 작성
- 이벤트 처리 함수에 네이티브 HTML이벤트 객체와 동일한 속성과 메서드를 갖는 이벤트 객체를 전달함
- 리액트는 브라우저의 네이티브 이벤트를 SyntheticEvent로 감싸서 사용하며 모든 브라우저에서 동일하게 작동함

---
# 화면 업데이트
- 컴포넌트에서 정보를 저장하고 싶을때 컴포넌트에 state 추가
## `useState`
1. 내장 hooks인 `userState` import 하기
```js
import {useState} from 'react';
```

2. 컴포넌트 내부에 상태 변수 선언하기
```js
function Mybutton() {
	const [count, setCount] = useState(0);
}
```
- 구조 분해시 `[something, setSomething]`형식으로 작성하는게 관례
- useState(0)이므로 count의 초기값은 0
- 상태를 변경하려면 `setSomething()`을 호출하고 새 값을 전달하면 됨

3. count값 변경하기
```js
function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);// count값 변경
  }

  return (
    <button onClick={handleClick}> // count값을 변경하는 함수를 이벤트 함수로 전달
      Clicked {count} times // 속성 값인 count를 사용
    </button>
  );
}
```
- 동일한 컴포넌트를 렌더링하면 각각 객체가 생성되어 자기들만의 상태를 가짐

---
# Hooks 사용
- Hooks: `use`로 시작하는 함수
- 기존 hooks을 조합해 직접 hooks 작성 가능
- 컴포넌트의 상단 또는 다른 hooks에서만 hooks를 호출할 수 있음
- 조건문이나 반복문에서 `useState`를 사용하려면 새로운 컴포넌트를 추출하고 그곳에 두면됨(잘 이해 안됨)

|                       |                                                                                                                                                                                                                            |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useState`            | 함수 컴포넌트에 리액트 상태를 추가할수 있게 함                                                                                                                                                                             |
| `useEffect`           | - 함수 컴포넌트에서 사이드 이펙트를 수행할 수 있게 해줌<br> - `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`와 같은 목적으로 사용됨                                                                     |
| `useContext`          | 중첩을 추가하지 않고도 리액트 컨텍스트에 구독할 수 있게 해줌                                                                                                                                                               |
| `useReducer`          | 복잡한 상태로직에서 자주 사용<br> 여러 하위 값이 관련되거나 다음 상태가 이전 상태에 의존하는 복잡한 상태로직에 사용                                                                                                        |
| `useCallback`         | 의존성 중 하나가 변경될때만 변경되는 메모이즈된 콜백 버전 반환<br> 불필요한 렌더링을 방지하기 위해 참조 동등성을 기반으로 최적화된 자식 컴포넌트에 콜백 전달할때 유용                                                      |
| `useMemo`             | 메모이즈된 값 반환<br> 생성 함수와 의존성 배열 전달<br> 의존성 중 하나가 변경될때만 메모이즈된 값을 다시 계산                                                                                                              |
| `useRef`              | 전달된 인수(initialValue)로 초기화된 `.current`속성을 갖는 가변적인 ref객체 생성<br> 반환된 객체는 컴포넌트의 전체 수명동안 유지됨                                                                                         |
| `useImperativeHandle` | ref를 사용할때 부모 컴포넌트에 노출되는 인스턴스 값을 사용자 정의 설정하는데 사용                                                                                                                                          |
| `useLayoutEffect`     | `useEffect`와 동일한 형태로 동작하지만 DOM 변경 후 동기적으로 발생<br> DOM에서 레이아웃을 읽고 동기적으로 다시 렌더링할 수 있음<br> `useLayoutEffect`내에서 예약된 업데이트는 브라우저가 그려지기 전에 동기적으로 플러시됨 |
|`useDebugValue`|react DevTools에서 사용자 정의 훅의 레이블을 표시하는데 사용됨|

---
# 컴포넌트 간 데이터 공유(상태 끌어올리기)
- 컴포넌트 간 데이터를 공유하려면 개별 컴포넌트에 상태를 두지 말고 모두를 포함하는 상위의 가장 가까운 컴포넌트에 상태를 두어 하위 컴포넌트에 전달하면 됨
- props: JSX 중괄호를 사용해 하위 컴포넌트로 전달하는 정보들
```js
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>함께 업데이트되는 카운터</h1>
      // onClick prop가 handleClick함수로 설정됨
      // count값이 prop로 전달됨
      <MyButton count={count} onClick={handleClick} /> 
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      {count}번 클릭됨
    </button>
  );
}
```

---
# React 문법

---
## App.js
- App.js에서 컴포넌트를 사용해야 로컬 주소를 브라우저로 열었을때 진짜 사용됨
## index.js
```js
const root = ReactDOM.createRoot(document.getElementById('root'));

root.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>
);

reportWebVitals();
```
- index.js는 App.js에서 생성한 컴포넌트를 웹 브라우저를 연결함
- `ReactDOM.createRoot()`로 index.html에서 id가 root인 DOM을 선택해 루트 돔 객체 생성
- `루트 돔 객체.render()`: 브라우저에 있는 실제 DOM내부에 컴포넌트를 렌더링하겠다는 의미
- `<App/>`은 App.js 컴포넌트임
## index.html
- 메인 페이지 개념
- index.js에서 `<div id="root"></div>`가 선택되어 이 태그 내부에 컴포넌트가 렌더링됨
---
## 컴포넌트
- 컴포넌트는 함수로 작성함
- `export default`된 컴포넌트는 함수명으로 다른 컴포넌트에서 HTML태그(JSX 요소)로 사용 가능
## 컴포넌트 함수에 붙이는 키워드
- `export`: 함수가 파일 외부에서 접근 가능하도록 만듬
- `default`: 코드를 사용하는 다른 파일에서 해당 함수가 주 파일임을 나타냄
## 함수 반환문
- `return`: 뒤에 오는 내용을 함수의 호출자에게 반환

---
## JSX 문법
- JSX는 자바스크립트 코드와 HTML태그의 조합
- `className=`: 버튼의 스타일을 지정하는 속성 또는 prop
- 모든 HTML 태그는 반드시 `/`로 닫아야함
- 태그와 태그 사이에 내용이 없을때는 self closing 태그를 사용해야함 e.g. `<hello/>`
- 2개 이상의 태그는 반드시 하나의 태그로 감싸져야함
	- 감싼 태그를 별도의 엘리먼트로 사용하기 싫다면 fragment로 감싸면 됨
```js
function App() {
	return (
		<> //fragment
			<Hello/>
		</>
	);
}
```
- JSX 내부에 자바스크립트 변수를 보여줄때는 변수를 중괄호(`{}`)로 감쌈
```js
function App() {
	return (
		<>
			<div>{name}</div>
		</>
	);
}
```
- 인라인 스타일은 컴포넌트내에 객체 형태로 작성해야함
- `-`로 구분된 스타일 이름은 카멜케이스로 작성해야함
```js
function App() {
	const name = 'react';
	const style = {
		backgroundColor: 'black',
		color: 'aqua',
		fontSize: 10,
		padding: '1rem'
	}

	return (
		<>
			<div style={style}>{name}</div>
		</>
	);
}
```
- 태그내에 설정하는 CSS class는 `className=`으로 설정해야함
```css
.gray-box {
	background: gray;
	width: 20px;
	height: 20px;
}
```

```js
function App() {
	return (
		<div className="gray-box"></div>
	)
}
```
- 태그 내에 지정하는 `key={}`는 컴포넌트의 식별자 역할임
- key를 이용해 리액트는 재렌더링 사이에 컴포넌트의 상태를 유지할 수 있음
- 컴포넌트의 키가 변경되면 해당 컴포넌트는 파괴되고 새로운 상태로 다시 생성됨
- 리액트는 요소가 생성될때 key 속성을 추출하고 반환된 요소에 직접 key를 저장함
- 리액트는 자동으로 key를 사용해 어떤 컴포넌트를 업데이트할지 결정함
- 컴포넌트가 부모로부터 어떤 키를 지정했는지 알 수 없음
- **동적인 리스트를 구축할때 적절한 key를 할당해야함**

- JSX 내부의 주석은 `{/* 이런 형태로 */}`작성, 중괄호로 감싸야 화면에 보이지 않음
- 열리는 태그 내부에서는 `//`로 주석 작성 가능
```js
function App() {
	return (
		{/* 화면에 보이지 않는 주석 */}
		<div
			// 열리는 태그 내부의 주석
		></div>
	)
}
```

---
## props
- properties의 줄임말로, 어떤 값을 컴포넌트에게 전달할때 사용(부모 컴포넌트에서 자식 컴포넌트로 데이터 전달)
- props는 함수에 전달되는 인수
- 이벤트를 나타내는 prop명은 `onSomething`이 관례
- 이벤트를 처리하는 함수명은 `handleSomething`이 관례
- 컴포넌트에 어떤 값을 전달해주려면 태그내에 `속성명=값`형식으로 작성
- 컴포넌트에게 전달되는 props는 해당 컴포넌트에서 파라미터로 조회 가능
- 전달되는 props는 객체형태이며, 지정한 속성값을 `파라미터명.속성명`형식으로 사용 가능
- props가 객체 형태이므로 함수 파라미터 부분에서 구조 분해를 사용하면 더 간결함
- `컴포넌트명.defaultProps = {속성명: 값,...}`: 컴포넌트에 props를 지정하지 않았을때 사용할 기본값 설정
- 태그내에 중괄호를 2번 사용하는 건 가장 바깥의 중괄호는 JSX문법, 안쪽 중괄호는 객체 리터럴임
> [!객체 리터럴]
> 객체 생성 방식 중 하나로, 일반적으로 많이쓰는 객체명 = {key: value, ...} 형식

> [!데이터 불변성의 장점]
> - 복잡한 기능을 구현하기 훨씬 쉬워짐
> - 데이터의 변경 여부 비교를 매우 저렴하게 만들어 컴포넌트 재렌더링을 시점을 정할 수 있음

```js
export default function Hello({color, name}) {
	return <div style={{color}}>안녕하세요 {name}</div>
}

Hello.defaultProps = {
	name: '이름없음'
}
```

```js
export default function App() {
	return (
		<>
			<Hello name="react" color="red"/>
			<Hello color="pink"/>
		</>
	);
}
```

- 컴포넌트 태그 사이에 넣은 값을 사용할때는 `props.children`사용
```js
export default function Wrapper({children}) {
	return (
		<div>{children}</div> 
		{/*props.children을 사용해야 App컴포넌트에서 Wrapper태그에 둘러싸인 컴포넌트 태그들이 화면에 표시됨*/}
	)
}
```

```js
export default function App() {
	return (
		<Wrapper>
			<Hello name="react" color="red"/>
			<Hello color="pink"/>
		</Wrapper>
	);
}
```

---
## state(`useState()`)
- 상태는 시간에 따라 변하는 데이터
- 시간이 지나도 값이 변경되지 않거나, 부모로부터 props를 통해 전달받거나, 컴포넌트 내에서 기존 상태나 props를 기반으로 계산할 수 있다면 상태가 아님
- 상태는 상호작용에만 사용됨
- 부모 컴포넌트는 상태를 하위 컴포넌트에 props로 전달할 수 있음
- 애플리케이션이 기억해야 하는 최소한의 변경 데이터 집합
- 상태를 구조화하는 가장 중요한 원칙은 DRY(Don't Repeat Yourself)를 유지하는 것
- 애플리케이션이 필요로하는 상태의 절대 최소한의 표현을 찾고 나머지는 필요할 때 계산하기