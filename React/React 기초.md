# React 특징
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
- 중괄호를 사용해 자바스크립트 표현식 삽입 가능
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
# App.js
- App.js에서 컴포넌트를 사용해야 로컬 주소를 브라우저로 열었을때 진짜 사용됨
# index.js
```js
const root = ReactDOM.createRoot(document.getElementById('root'));

root.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>
);

reportWebVitals();
```
- `ReactDOM.createRoot()`에서 id가 root인 DOM을 선택해 루트 돔 객체 생성
- `루트 돔 객체.render()`: 브라우저에 있는 실제 DOM내부에 컴포넌트를 렌더링하겠다는 의미
- `<App/>`은 App.js 컴포넌트임
# index.html
- 메인 페이지 개념
- index.js에서 `<div id="root"></div>`가 선택되어 이 태그 내부에 컴포넌트가 렌더링됨
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
- 특별한 구문 없이 자바스크립트의 조건문 활용
- 속성을 조건부로 지정하는 데에도 동일하게 적용

---
# 목록 렌더링


---
# 이벤트 처리


---
# 상태 관리


---
# 상태 끌어올리기


---

