# 라우팅이란?
- 라우팅: 웹 애플리케이션에서 라우팅은 사용자가 요청한 URL에 따라 알맞은 페이지를 보여주는 것
- react-router는 라우팅을 간단하게 해주고, 서버 사이드 렌더링을 도와주는 도구도 함께 제공
- react-router를 사용하면 클라이언트 사이드, 즉 브라우저측에서 js를 사용해 라우트를 관리하게 되므로 검색 잘 안되고, js파일이 아직 캐싱되지 않으면 흰 페이지가 나타날 수 있음
```
yarn add react-router-dom
```
# react-router 사용법
1. index.js의 `<App />`을 `<BrowserRouter>`로 감싸기
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom'; // * BrowserRouter 불러오기
import App from './App';

// * App 을 BrowserRouter 로 감싸기
ReactDOM.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById('root')
);
```

2. 특정 주소에 컴포넌트 연결하기
```jsx
<Route path="주소규칙" element={보여 줄 컴포넌트 JSX} />
```
- `<Route>`는 `<Routes>`안에 있어야함
```jsx
import React from 'react';
import { Route, Routes } from 'react-router-dom';
import About from './About';
import Home from './Home';

const App = () => {
  return (
    <Routes>
      <Route path="/" component={<Home />} />
      <Route path="/about" component={<About />} />
    </Routes>
  );
};

export default App;
```

3. `<Link>`로 다른 주소로 이동하기
	- react-router를 사용할때는 `<a>`태그를 사용하면 안되는데, `<a>`태그는 기본적으로 페이지를 이동시키면서 페이지를 새로 불러와 새로 렌더링되기 때문
	- `<a>`태그 사용하려면 onClick에 `e.preventDefault()` 호출하고 따로 자바스크립트로 주소 변환시켜야함
	- `<Link>`는 HTML5 History API를 사용해 브라우저의 주소만 바꾸고 페이지를 새로 불러오지 않음
	- 사용법: `<Link to="/">home</Link>`
# URL 파라미터 사용하기
- URL 파라미터는 주로 id나 이름으로 조회할때 사용
- `/.../:파라미터`로 경로 설정
- 파라미터가 여러 개면 `/.../:파라미터1/:파라미터2`
```jsx
const App = () => {
  return (
    <Routes>
      <Route path="/" exact={true} component={<Home />} />
      <Route path="/about" component={<About />} />
      <Route path="/profiles/:username" component={<Profile />} />
    </Routes>
  );
};

export default App;
```
- `useParams()`훅으로 URL파라미터를 객체 형태로 가져옴
```jsx
import { useParams } from 'react-router-dom';

const data = {
  gildong: {
    name: '홍길동',
    description: '고전 소설 홍길동전의 주인공',
  },
};

const Profile = () => {
  const params = useParaams();
  const profile = data[parmas.username]; //URL파라미터 값 가져옴
  return (
    <div>
      {profile ? (
        <h3>
          {username}({profile.name})
        </h3>
        <p>{profile.description}</p>
      ) : (
        <p>존재하지 않는 프로필입니다.</p>
      )}      
    </div>
  );
};

export default Profile;
```
# 쿼리스트링 사용하기
- 쿼리스트링은 주로 키워드 검색, 페이지네이션, 정렬 방식 등 데이터 조회에 필요한 옵션을 전달할때 사용
- `useLocation`훅은 현재 페이지의 정보를 가진 location객체를 가짐
	- location객체에 있는 값들
		- pathname: 쿼리스트링 제외한 현재 주소 경로
		- search: 맨 앞의 `?`문자 포함한 **쿼리스트링 값**
		- hash: 주소의 `#`문자열 뒤의 값
		- state: 페이지로 이동할때 임의로 넣을 수 있는 상태 값
		- key: location객체의 고유값으로, 초기에는 default였다가 페이지 바뀔때마다 고유 값 생성됨
## useSearchParams
- react-router-dom v6 이상부터는 `useSearchParams`사용해 쿼리스트링 가져올 수 있음
- 배열 타입의 값 반환, 첫번째 원소를 `get`으로 쿼리파라미터를 조회하거나 `set`으로 수정 가능, 두번째 원소는 쿼리파라미터를 객체형태로 업데이트하는 setter
> [!warning] 쿼리파라미터를 파싱한 값은 문자열임!

```jsx
import { useSearchParams } from 'react-router-dom';

const About = () => {
  const [searchParams, setSearchParams] = useSearchParams();
  const detail = searchParams.get('detail');
  const mode = searchParams.get('mode');

  const onToggleDetail = () => {
    setSearchParams({ mode, detail: detail === 'true' ? false : true });
  };

  const onIncreaseMode = () => {
    const nextMode = mode === null ? 1 : parseInt(mode) + 1;
    setSearchParams({ mode: nextMode, detail });
  };

  return (
    <div>
      <h1>소개</h1>
      <p>리액트 라우터를 사용해 보는 프로젝트입니다.</p>
      <p>detail: {detail}</p>
      <p>mode: {mode}</p>
      <button onClick={onToggleDetail}>Toggle detail</button>
      <button onClick={onIncreaseMode}>mode + 1</button>
    </div>
  );
};

export default About;
```
- react-router-dom v6 아래는 qs 라이브러리 사용
```jsx
import React from 'react';
import qs from 'qs';

const About = () => {
  const query = qs.parse(location.search, {
    ignoreQueryPrefix: true
  });
  const detail = query.detail === 'true'; // 쿼리의 파싱결과값은 문자열

  return (
    <div>
      <h1>소개</h1>
      <p>이 프로젝트는 리액트 라우터 기초를 실습해보는 예제 프로젝트랍니다.</p>
      {detail && <p>추가적인 정보가 어쩌고 저쩌고..</p>}
    </div>
  );
};

export default About;
```
## useParams
- Route path에 지정한 최근 URL의 파라미터 반환
	- '.../grammarbook/1'에서 grammarbook은 pathname, 1이 파라미터
```jsx
import { useParams } from 'react-router-dom';

const test = () => {
  let {params} = useParams();

  return <p>현재 페이지의 파라미터: {params}</p>;
}
```
# 중첩된 라우트
- 중첩된 라우트는 상위 경로가 같은 라우트를 해당 상위 라우트 안에 넣어, 상위 페이지의 컴포넌트가 하위 페이지에서도 보임
- `<Outlet />`: 상위 페이지의 컴포넌트에 children으로 들어가는 컴포넌트
```js
const App = () => {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/profiles/:username" element={<Profile />} />
      <Route path="/articles" element={<Articles />}>
        <Route path=":id" element={<Article />} /> //중첩된 라우트
      </Route>
    </Routes>
  );
};

export default App;
```

```jsx
import { Link, Outlet } from 'react-router-dom';

const Articles = () => {
  return (
    <div>
      <Outlet /> {/*하위 페이지의 컴포넌트가 들어감*/}
      <ul>
        <li>
          <Link to="/articles/1">게시글 1</Link>
        </li>
        <li>
          <Link to="/articles/2">게시글 2</Link>
        </li>
        <li>
          <Link to="/articles/3">게시글 3</Link>
        </li>
      </ul>
    </div>
  );
};

export default Articles;
```
## 공통 레이아웃 컴포넌트
- 중첩된 라우트를 활용하면 컴포넌트를 한번만 사용해서 여러 페이지에 보여줄 수 있음
```jsx
const App = () => {
  return (
    <Routes>
      <Route element={<Layout />}>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/profiles/:username" element={<Profile />} />
      </Route>
    </Routes>
  );
};

export default App;
```

```jsx
import { Outlet } from 'react-router-dom';

const Layout = () => {
  return (
    <div>
      <header style={{ background: 'lightgray', padding: 16}}>
        Header
      </header>
      <main>
        <Outlet />
      </main>
    </div>
  );
};

export default Layout;
```
## index props
- `<Route index ... />`: 
- `index` == `path="/"`
# 부가기능
## useNavigate
> [!warning] react-router v5의 `useHistory`가 v6에서 `useNavigate`로 변경됨

- `<Link>`를 사용하지 않고 다른 페이지로 이동
- `navigate(숫자)`: 페이지 앞/뒤로 이동
- `navigate(..., {replace: true})`: 현재 페이지를 기록에 남기지 않음
```jsx
import { Outlet, useNavigate } from 'react-router-dom';

const Layout = () => {
  const navigate = useNavigate();

  const goBack = () => {
    // 이전 페이지로 이동
    navigate(-1);
  };

  const goArticles = () => {
    // articles 경로로 이동
    navigate('/articles');
  };

  return (
    <div>
      <header style={{ background: 'lightgray', padding: 16, fontSize: 24 }}>
        <button onClick={goBack}>뒤로가기</button>
        <button onClick={goArticles}>게시글 목록</button>
      </header>
      <main>
        <Outlet />
      </main>
    </div>
  );
};

export default Layout;
```
## NavLink
- 링크에서 사용하는 경로가 현재 라우트의 경로와 일치하면 특정 스타일이나 css클래스 적용하는 컴포넌트
- boolean값 `isActive`를 파라미터로 받는 함수 타입의 값을 style이나 className속성에 전달​​​​​​​​​​​​​​​​
```jsx
import { NavLink, Outlet } from 'react-router-dom';

const Articles = () => {
  const activeStyle = {
    color: 'green',
    fontSize: 21,
  };
  return (
	<div>
	<Outlet />
	<NavLink
	  to="/articles/1"
	  style={({isActive}) => (isActive ? activeStyle : undefined)}
    >
      게시물 1
	</NavLink>
	</div>
  );
}
```
## NotFound 페이지 만들기
- `<Route path="*" element={<NotFound />} />`
- `*`는 모든 라우트 규칙(경로)에서 일치하는 페이지가 없다면 해당 라우트가 화면에 나타남
## Navigate 컴포넌트
- 화면에 컴포넌트를 보여주는 순간 다른 페이지로 이동(페이지 리다이렉트)할때 사용
	e.g. 로그인이 필요한 페이지를 로그인 안한 상태로 접속했을때 로그인 페이지로 이동
```jsx
import { Navigate } from 'react-router-dom';

const MyPage = () => {
  const isLoggedIn = false; //로그인 유무 나타내는 가상의 변수

  if (!isLoggedIn) {
    return <Navigate to="/login" replace={true} />;
  }

  return <div>마이 페이지</div>;
};

export default MyPage;
```
