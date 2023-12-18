# React 특징
---
- 리액트 자체는 라이브러리
- 리액트 기반의 프레임워크: Next.js, Remix, Gatsby 등
## 컴포넌트 기반
---
- React 코드는 컴포넌트라는 개체로 구성
- 컴포넌트는 썸네일, 좋아요 버튼, 비디오 등 개별적인 조각 또는 전체 페이지처럼 UI들을 통칭함
- 컴포넌트는 JSX 문법을 사용해 마크업(html태그)을 반환하는 자바스크립트 클래스나 함수임
- JSX와 리액트는 서로 다른 것임(JSX는 구문 확장, 리액트는 자바스크립트 라이브러리)
- 컴포넌트 내에서 이벤트 핸들러 함수 정의
- 컴포넌트를 생성하고 이들을 결합해 사용자 인터페이스 구성
## 선언적 프로그래밍
---
- 리액트는 UI가 어떻게 구성되거나 업데이트되는지가 아니라 어떻게 보여져야 하는지를 설명하는 코드를 작성하도록 장려
- 뷰가 데이터 변경에 자동으로 맞춰지게 함
- 데이터가 변경될때 React가 적절한 컴포넌트를 업데이트하고 렌더링함
- 코드를 더 가독성있게 만들어 디버깅이 쉬워짐
## 가상 DOM
---
- React는 실제 DOM의 가벼운 복사본인 가상 DOM을 생성함
- 조정(reconciliation): 객체의 상태가 변경되면 React는 가상 DOM에 작업 후 이전 가상 DOM과 새로운 가상 DOM의 차이 비교해 실제 DOM에서 실제로 변경된 부분만 업데이트
- 컴포넌트 --> 가상 DOM --> 실제 DOM --> UI 렌더링
  --> 이벤트에 따른 렌더링: 이벤트 처리 --> 상태 관리 --> 데이터 흐름 --> 단방향 데이터 흐름 --> 컴포넌트 갱신 --> 재렌더링
## 단방향 데이터 흐름과 FLUX
---
- React는 단방향 데이터 흐름을 구현해 애플리케이션을 이해하기 쉽게 만듬
- FLUX는 데이터의 단방향 유지를 돕는 패턴
- React 앱 설계시 종종 부모 컴포넌트 내에 자식 컴포넌트를 중첩시켜 FLUX 구현
## 라우팅과 전역 상태 관리
---
- React 16.8 이후로 Hooks 기능을 통해 함수형 컴포넌트에서도 상태 관리 가능
- Redux 같은 추가적인 라이브러리로도 상태관리 가능
- React-Router로 라우팅 기능 사용
## JSX
---
- 자바스크립트 확장 마크업
- HTML보다 엄격하며 모든 태그를 닫아야함
- 스타일링을 위해 `className`속성을 사용해 CSS 클래스 지정 가능
- 중괄호를 사용해 자바스크립트 표현식(변수 등) 삽입 가능
  --> 자바스크립트 코드의 변수를 기반으로 동적으로 데이터 표시
- JSX를 관련 렌더링 로직 근처에 배치하면 컴포넌트를 쉽게 생성, 유지, 삭제할 수 있음
## 필요한 곳마다 상호작용성 추가
---
- 컴포넌트는 데이터를 받아들이고 화면에 표시해야할 내용 반환
- 사용자가 입력란에 입력하는 등의 상호작용에 대해 새로운 데이터를 컴포넌트에 전달하면 React는 화면을 새로운 데이터와 일치하도록 업데이트함
- 페이지 전체를 React로 구축할 필요 없이 기존의 HTML페이지에 React를 추가하고 인터랙티브한 컴포넌트를 어디서든 렌더링할 수 있음

# React 프로젝트 생성하기
---
1. 터미널에 `npx create-react-app [프로젝트 폴더명]`입력
	- [[설치 관련]](프로젝트 생성시 오류 참고)
2. `npm start`입력후 http://localhost:3000/ 열리는지 확인

# React 기본 사용법
---
## App.js
---
- App.js에서 컴포넌트를 사용해야 로컬 주소를 브라우저로 열었을때 진짜 사용됨
## index.js
---
```jsx
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
---
- 메인 페이지 개념
- index.js에서 `<div id="root"></div>`가 선택되어 이 태그 내부에 컴포넌트가 렌더링됨

## 컴포넌트
---
- **컴포넌트 이름은 항상 대문자로 시작**
- 순수 HTML 태그는 항상 소문자로 시작
- 컴포넌트 렌더링 중에는 prop, state, context 입력만 읽을 수 있고, 이들은 항상 읽기 전용으로 취급해야함
- 사용자 입력에 응답하여 무언가를 변경하려면 변수 대신 state를 설정해야함
- 컴포넌트내에서 다른 컴포넌트를 정의하면 매우 느려지고 버그 발생
  --> 모든 컴포넌트를 최상위 수준에서 정의하기
```jsx
export default function Gallery() {
  // 🔴 절대로 다른 컴포넌트 내에서 컴포넌트를 정의하지 마세요!
  function Profile() {
    // ...
  }
  // ...
}
```

```jsx
export default function Gallery() {
  // ...
}

// ✅ 최상위 수준에서 컴포넌트를 선언하세요
function Profile() {
  // ...
}
```
- `export default`: 해당 컴포넌트를 내보내는 키워드, 하나의 파일에 하나만 지정 가능(표준 자바스크립트 구문임)
- 명명된 내보내기는 하나의 파일에 여러 개 지정 가능
> default와 named export 중 하나만 사용하거나 하나의 파일에 같이 사용하지 않는게 좋음

|               | 내보내기 구문                         | 가져오기 구문                       |
| ------------- | ------------------------------------- | ----------------------------------- |
| 기본 내보내기 | `export default function Button() {}` | `import Button from './Button.js';`<br>import 뒤에 아무 이름 사용해도됨 |
| 명명된 내보내기|`export function Button() {}`|`import {Button} from './Button.js';`<br>import 뒤의 이름이 가져오려는 컴포넌트와 일치해야함|

- 컴포넌트는 여러 개의 JSX 태그를 반환할 수 없음
	- 여러 개의 JSX 태그를 공유 부모(`<div>...</div>나 <>...</>`)로 묶어야함
	- JSX로 변환할 HTML이 많다면 [온라인 변환 도구](https://transform.tools/html-to-jsx)사용
```jsx
function AboutPage() {
  return (
    <>
      <h1>About</h1>
      <p>Hello there.<br />How do you do?</p>
    </>
  );
}
```
### 컴포넌트의 순수성 유지하기
---
- 컴포넌트를 엄격하게 순수 자바스크립트 함수로 작성하면 코드베이스가 커져도 버그와 예측할 수 없는 동작을 피할 수 있음
- 순수 함수:
	- 자신의 일에만 전념함
	- 함수 호출전에 존재하는 객체나 변수를 변경하지 않음
	- 같은 입력에 대해 항상 같은 출력을 반환함
- 리액트는 작성하는 모든 컴포넌트가 순수 함수라고 가정함
- 로컬 변이: 컴포넌트 외부에 존재하는 객체나 변수를 변경하지 않고 내부에 있는 것만 변경할 수 있음
- 일반적으로 컴포넌트가 렌더링되는 순서에 의존하지 않아야함
- 이벤트 핸들러는 side effect의 일종이므로 순수할 필요 없음
- Strict 모드는 컴포넌트 함수를 두번 호출해 순수성을 어긴 컴포넌트를 찾을 수 있음(순수 함수는 두번 호출해도 결과가 같음)
	- `<React.StrictMode>`로 루트 컴포넌트 감싸서 Strict 모드 실행
	  e.g. 기본적으로 생성되는 리액트 앱의 index.js에 있음
	- Strict 모드는 개발시에만 실행되므로 프로덕션 걱정 안해도됨
```jsx
let guest = 0;

function Cup() {
	//Bad: 렌더링 이전에 존재한 변수 guset를 변경함
	guest = guest + 1;
	return <p>Tea cup for guest #{guest}</p>;
}
```
위의 경우는 `guest`변수를 `Cup`컴포넌트의 prop로 만들어서 해결할 수 있음

- 하나의 순수한 컴포넌트는 여러 사용자 요청에 대해 서비스 제공 가능
- 입력이 동일한 컴포넌트는 캐쉬된 컴포넌트를 사용할 수 있음
- 깊은 컴포넌트 트리를 렌더링하는 도중에 데이터가 변경되면 리액트는 구식 렌더링을 완료하기 전에 렌더링 재시작함. 순수성으로 인해 언제든지 계산을 중단해도 안전하기 때문

## JSX 문법
---
- JSX는 자바스크립트 코드와 HTML태그의 조합
- 모든 것을 카멜케이스로 작성해야함(`aria-*`, `data-*`속성은 관행적으로 `-`사용)
- 모든 HTML 태그는 반드시 `/`로 닫아야함
- 태그와 태그 사이에 내용이 없을때는 self closing 태그를 사용해야함 e.g. `<Hello/>`
- 2개 이상의 태그는 반드시 하나의 태그로 감싸져야함
	- 감싼 태그를 별도의 엘리먼트로 사용하기 싫다면 fragment로 감싸면 됨
```jsx
function App() {
	return (
		<> //fragment
			<Hello/>
		</>
	);
}
```
- JSX 내부에 중괄호`{}`로 자바스크립트 표현식을 넣을 수 있음
```jsx
function App() {
	return (
		<>
			<div>{name}</div>
		</>
	);
}
```
- 인라인 스타일은 JSX 요소의 style 속성에 객체 형태로 작성해야함
```jsx
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
- JSX에서 중괄호 안에 있는게 자바스크립트 객체일때 중괄호가 2번`{{ }}` 사용됨
> [!객체 리터럴]
> 객체 생성 방식 중 하나로, 일반적으로 많이쓰는 객체명 = {key: value, ...} 형식
- 태그내에 설정하는 CSS class는 `className=`으로 설정
- CSS파일 적용하려면 컴포넌트 파일의 상단에 `import [css파일 경로]`추가
```css
.gray-box {
	background: gray;
	width: 20px;
	height: 20px;
}
```

```jsx
import './MyComponent.css';  // CSS를 import

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
```jsx
function App() {
	return (
		{/* 화면에 보이지 않는 주석 */}
		<div
			// 열리는 태그 내부의 주석
		></div>
	)
}
```

## props
---
- properties의 줄임말로, 부모 컴포넌트에서 자식 컴포넌트로 전달되는 데이터(인수)
- 이벤트를 나타내는 prop명은 `onSomething`이 관례
- 이벤트를 처리하는 함수명은 `handleSomething`이 관례
- 컴포넌트에 어떤 값을 전달해주려면 태그내에 `속성명=값`형식으로 작성
- 컴포넌트에게 전달되는 props는 해당 컴포넌트 함수의 파라미터로 들어감
- 전달되는 props는 객체형태이며, 지정한 속성값을 `파라미터명.속성명`형식으로 사용 가능
- props가 객체 형태이므로 함수 파라미터 부분에서 구조 분해 사용 가능
- 스프레드 구문`{...props}`이 거의 모든 컴포넌트에서 사용된다면 문제있을 수 있음
### prop 기본값 설정
---
- `컴포넌트명.defaultProps = {속성명: 값,...}`
- 컴포넌트 파라미터에서 `{..., 속성명=값, ...}`
- 기본값이 null이나 0이면 사용되지 않음
```jsx
export default function Hello({color, name}) {
	return <div style={{color}}>안녕하세요 {name}</div>
}

Hello.defaultProps = {
	name: '이름없음'
}
```

```jsx
export default function App() {
	return (
		<>
			<Hello name="react" color="red"/>
			<Hello color="pink"/>
		</>
	);
}
```

### 자식으로 JSX 전달하기
---
- 컴포넌트 태그 사이에 태그를 중첩할때는 `props.children`사용
- 부모 컴포넌트는 중첩된 JSX 태그를 `children`prop로 받음
- `children` prop는 부모 컴포넌트에 의해 임의의 JSX로 채워질 수 있는 빈공간이라고 생각하면됨
```jsx
export default function Wrapper({children}) {
	return (
		<div>{children}</div> 
		{/*props.children을 사용해야 App컴포넌트에서 Wrapper태그에 둘러싸인 컴포넌트 태그들이 화면에 표시됨*/}
	)
}
```

```jsx
export default function App() {
	return (
		<Wrapper>
			<Hello name="react" color="red"/>
			<Hello color="pink"/>
		</Wrapper>
	);
}
```

### props를 변경하려고 하지 마라
---
- props는 불변임
- 컴포넌트는 시간에따라 다른 props를 받을뿐, 컴포넌트 내에서 props가 변경되면 안됨
- 사용자 상호 작용등으로 다른 props가 전달되면 이전 props는 버려지고, 자바스크립트 엔진이 메모리 회수함
- 사용자 상호 작용에 응답해야 할때는 **상태를 설정**해야함
> [!데이터 불변성의 장점]
> - 복잡한 기능을 구현하기 훨씬 쉬워짐
> - 데이터의 변경 여부 비교를 매우 저렴하게 만들어 컴포넌트 재렌더링을 시점을 정할 수 있음

## 조건부 렌더링
---
- 특별한 구문 없이 자바스크립트의 조건문(`if`문, 삼항 연산자, `&&`연산자 등) 활용
- 속성을 조건부로 지정하는것도 조건문 활용하면 됨
- 너무 많은 조건부 마크업으로 코드가 복잡해지면 자식 컴포넌트 추출 고려
### JSX를 조건부로 포함하기
---
```jsx
//if문 활용
function Item({name, isPacked}) {
	if (isPacked) {
		return <li className="item">{name} v</li>;
	}
	return <li className="item">{name}</li>;
}

```
- `{cond ? <A/> : <B/>`: 만약 cond면 `<A/>`를 렌더링하고, 그렇지 않으면 `<B/>`를 렌더링함
```jsx
//삼항연산자 활용
function Item({name, isPacked}) {
	return (
		<li className="item">
			{isPacked ? (
				<del>
					{name + 'v'}
				</del>
			) : (name)}
		</li>
	);
}
```
- `{cond && <A/>`: 만약 cond면 `<A/>`를 렌더링하고, 그렇지 않으면 아무것도 렌더링하지 않음
- JSX 요소는 내부 상태를 가지지 않고 실제 DOM 노드가 아닌 가벼운 설명(청사진)이기 때문에 if문을 활용한 방식과 완전히 동일함
```jsx
//&&연산자 활용
function Item({name, isPacked}) {
	return (
		<li className="item">
			{name} {isPacked && 'v'} //isPacked가 true면 v 렌더링
		</li>
	);
}
```
- `&&`의 왼쪽 조건을 기준으로 오른쪽 요소를 렌더링할지 판단함
	- 왼쪽이 숫자면 전체 표현식은 해당 값이 되고, 리액트는 숫자 자체를 렌더링하므로 왼쪽을 조건식으로 만들어야함
### 변수를 조건부로 할당하기
---
```jsx
function Item({name, isPacked}) {
	let itemContent = name;
	if (isPacked) {
		itemContent = name + 'v';
	}
	return (
		<li className="item">
			{itemContent}
		</li>
	);
}
```

## 목록 렌더링
---
- `for`문 또는 `배열.map()`, `배열.filter()` 사용
```jsx
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
}, ...];
```
### `map()`
---
- 배열에 든 원소들을 JSX요소로 변환하여 반환
- `배열객체.map(배열원소 => JSX요소)`
- `<Fragment></Fragment>`: 한 항목마다 2개 이상의 DOM 노드를 렌더링해야하는 경우 사용. Fragment 태그는 DOM에서 사라짐(`<></>`와 유사)
```jsx
function List() {
	const listItems = people.map(person => 
		<li>{person}</li>
	);
	const manyNodesListItems = people.map(person => 
		<Fragment>
			<h3>{person.name}</h3>
			<p>{person.profession}</p>
		</Fragment>
	);
	return <ul>{listItems}</ul>;
}
```
### `filter()`
---
- 배열에 든 원소들 중 조건 함수 또는 식에 해당되는 원소들만 있는 배열 반환하므로 `map()`을 사용해 JSX 요소로 변환해야함
- `배열객체.filter(배열원소 => 조건함수 또는 식)`
```jsx
function List() {
	const chemists = people.filter(person =>
		person.profession === 'chemist'
	);
	const listItems = chemists.map(chemist => 
		<li>
			<b>{chemist.name}</b>
			{' ' + chemist.profession + ' '}
			known for {chemist.accomplishment}
		</li>
	);
}
```
### `key`로 목록 항목 순서 유지하기
---
- `map()`호출 내에서 JSX 요소는 항상 `key`가 필요함
- key는 목록의 각 항목을 고유하게 식별하는 값(일반적으로 데이터베이스 id값)
- 자바스크립트 배열의 인덱스를 key로 사용하지 말것
- key를 동적으로 생성하지 말것: key를 동적으로 생성하면 키가 렌더링 사이에 일치하지 않아 모든 컴포넌트와 DOM이 매번 재생성됨
- 컴포넌트는 key를 prop으로 받지 않음
- 컴포넌트가 id를 필요로하는 경우 별도의 prop으로 전달해야함 
  e.g. `<Profile key={id} userId={id}/>`
```jsx
const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```


