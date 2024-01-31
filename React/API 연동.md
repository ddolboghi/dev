# axios로 REST API 요청하기
- 설치
```
yarn add axios
```
- `useEffect`에 첫번째 파라미터로 등록하는 함수에는 `async` 사용 불가 -> 내부에 `async`사용하는 새로운 함수 선언
```jsx
function Users() {
  const [users, setUsers] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        // 요청이 시작 할 때에는 error 와 users 를 초기화하고
        setError(null);
        setUsers(null);
        // loading 상태를 true 로 바꿉니다.
        setLoading(true);
        const response = await axios.get(
          'https://jsonplaceholder.typicode.com/users'
        );
        setUsers(response.data);
      } catch (e) {
        setError(e);
      }
      setLoading(false);
    };

    fetchUsers();
  }, []);

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  if (!users) return null;
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.username} ({user.name})
        </li>
      ))}
    </ul>
  );
}

export default Users;
```
- `useEffect`내부에 있는 API 요청 함수 부분을 밖으로 빼내고 컴포넌트에 연결하면 해당 컴포넌트 동작에 따라 요청 보낼 수 있음
```jsx
function Users() {
  const [users, setUsers] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const fetchUsers = async () => {
      try {
        // 요청이 시작 할 때에는 error 와 users 를 초기화하고
        setError(null);
        setUsers(null);
        // loading 상태를 true 로 바꿉니다.
        setLoading(true);
        const response = await axios.get(
          'https://jsonplaceholder.typicode.com/users'
        );
        setUsers(response.data);
      } catch (e) {
        setError(e);
      }
      setLoading(false);
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  if (!users) return null;
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.username} ({user.name})
        </li>
      ))}
    </ul>
    <button onClick={fetchUsers}>다시 불러오기</button>
  );
}

export default Users;
```
# useReducer로 요청 상태 관리하기
- `useReducer`로 loading, success, error 액션에 따라 다르게 처리
- `useState`의 setter를 여러번 사용하지 않아도 되며, 로직을 분리해 재사용 쉬움
- 아래 코드 설명
	1. fetch함수에서 dispatch함수에 action.type에따라 동작 지정
	2. reducer가 action.type에따라 각 state(loading, data, error)를 지정후 반환
	3. useReducer의 state를 spread문법으로 펼쳐 단순 변수명만 data에서 users로 대체함
```jsx
import React, { useEffect, useReducer } from 'react';
import axios from 'axios';

//2
function reducer(state, action) {
  switch (action.type) {
    case 'LOADING':
      return {
        loading: true,
        data: null,
        error: null
      };
    case 'SUCCESS':
      return {
        loading: false,
        data: action.data,
        error: null
      };
    case 'ERROR':
      return {
        loading: false,
        data: null,
        error: action.error
      };
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

function Users() {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    data: null,
    error: null
  });
  
  //1
  const fetchUsers = async () => {
    dispatch({ type: 'LOADING' });
    try {
      const response = await axios.get(
        'https://jsonplaceholder.typicode.com/users'
      );
      dispatch({ type: 'SUCCESS', data: response.data });
    } catch (e) {
      dispatch({ type: 'ERROR', error: e });
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  //3. state.data를 users 키워드로 조회
  const { loading, data: users, error } = state; 

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  if (!users) return null;
  return (
    <>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={fetchUsers}>다시 불러오기</button>
    </>
  );
}

export default Users;
```
# useAsync 커스텀 훅
- `useAsync(API요청 콜백 함수, useEffect에서 사용하는 의존성 배열)`
- return: state값(아래 코드에서 loading, data, error가 포함된 객체), fetchData함수
	- fetchData함수는 API 재요청할때도 사용 가능
```jsx
import { useReducer, useEffect } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'LOADING':
      return {
        loading: true,
        data: null,
        error: null
      };
    case 'SUCCESS':
      return {
        loading: false,
        data: action.data,
        error: null
      };
    case 'ERROR':
      return {
        loading: false,
        data: null,
        error: action.error
      };
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

function useAsync(callback, deps = []) {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    data: null,
    error: false
  });

  const fetchData = async () => {
    dispatch({ type: 'LOADING' });
    try {
      const data = await callback();
      dispatch({ type: 'SUCCESS', data });
    } catch (e) {
      dispatch({ type: 'ERROR', error: e });
    }
  };

  useEffect(() => {
    fetchData();
  }, deps);

  return [state, fetchData];
}

export default useAsync;
```
위의 useAsync에서 Primise 결과를 data에 바로 담기 때문에 useAsync를 사용할때 요청 후 진짜 사용할 값을 추출해 반환하는 함수를 따로 만들어야함
```jsx
async function getUsers() {
  const response = await axios.get(
    'https://jsonplaceholder.typicode.com/users'
  );
  return response.data;
}

function Users() {
  const [state, refetch] = useAsync(getUsers, []);

  const { loading, data: users, error } = state; // state.data 를 users 키워드로 조회
...
}
```
- 특정 동작에서만 API 요청하고 싶으면
1. useAsync에 파라미터 추가
2. 내부 useEffect에 해당 파라미터로 할 동작 지정
```jsx
...
function useAsync(callback, deps = [], skip = false) {
  const [state, dispatch] = useReducer(reducer, {
    loading: false,
    data: null,
    error: false
  });

  const fetchData = async () => {
    dispatch({ type: 'LOADING' });
    try {
      const data = await callback();
      dispatch({ type: 'SUCCESS', data });
    } catch (e) {
      dispatch({ type: 'ERROR', error: e });
    }
  };

  useEffect(() => {
    if (skip) return; //아무 작업도 하지 않음
    fetchData();
  }, deps);
  return [state, fetchData];
}

export default useAsync;
```
## API에 파라미터가 필요한 경우
- URI: `.../${파라미터명}`
- 필요한 파라미터를 상위 컴포넌트에서 props로 받아오기
- useAsync가 받는 props는 당연히 useEffect의 의존성 배열에 추가해야함
```jsx
...
function Users() {
  const [userId, setUserId] = useState(null);
  const [state, refetch] = useAsync(getUsers, [], true);

  const { loading, data: users, error } = state;

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  if (!users) return <button onClick={refetch}>불러오기</button>;
  return (
    <>
      <ul>
        {users.map(user => (
          <li
            key={user.id}
            onClick={() => setUserId(user.id)} //id값을 state로 관리
            style={{ cursor: 'pointer' }}
          >
            {user.username} ({user.name})
          </li>
        ))}
      </ul>
      <button onClick={refetch}>다시 불러오기</button>
      {userId && <User id={userId} />} //User컴포넌트에 userId값을 넘겨줌
    </>
  );
}

export default Users;
```

```jsx
async function getUser(id) {
  const response = await axios.get(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );
  return response.data;
}

function User({ id }) {
  const [state] = useAsync(() => getUser(id), [id]);
  const { loading, data: user, error } = state;

  if (loading) return <div>로딩중..</div>;
  if (error) return <div>에러가 발생했습니다</div>;
  if (!user) return null;

  return (
    <div>
      <h2>{user.username}</h2>
      <p>
        <b>Email:</b> {user.email}
      </p>
    </div>
  );
}
```
# react-async 라이브러리
> [!info]
> - 오랫동안 유지보수하게될 프로젝트라면 커스텀 useAsync훅 사용하는게 좋음
> - 22.8.8) react 18 upgrade 버전에서 `<React.StrictMode />`를 제거해야 정상 작동
- useAsync 커스텀 훅과 비슷하지만 객체를 받고 반환함
- Promise를 반환하는 함수의 파라미터를 객체 형태로 해줘야함
- `watch`: useAsync의 watch에 특정 변수를 지정하면 해당 값이 바뀔때마다 promiseFn에 넣은 함수를 다시 호출해줌, 즉 **의존성 배열에 추가할 값임**
- `reload`: 커스텀 useAsync의 fetchData함수와 같은 역할(데이터 다시 불러오기)
- 특정 동작에서만 API 호출하려면 `reload` 대신 `run`, `promiseFn` 대신 `deferFn` 사용
```jsx
//렌더링할때마다 API 요청
const {data: users, error, isLoding, reload} = useAsync({
  promiseFn: getUser,
  id, 
  watch: id
})
```

```jsx
//특정 동작에서만 API 요청
const {data: users, error, isLoding, run} = useAsync({
  deferFn: getUser,
  id, 
  watch: id
})
```

```jsx
import { useAsync } from "react-async"

const loadCustomer = async ({ customerId }, { signal }) => {
  const res = await fetch(`/api/customers/${customerId}`, { signal })
  if (!res.ok) throw new Error(res)
  return res.json()
}

const MyComponent = () => {
  const { data, error, isLoading } = useAsync({ promiseFn: loadCustomer, customerId: 1 })
  if (isLoading) return "Loading..."
  if (error) return `Something went wrong: ${error.message}`
  if (data)
    return (
      <div>
        <strong>Loaded some data:</strong>
        <pre>{JSON.stringify(data, null, 2)}</pre>
      </div>
    )
  return null
}
```
# Context와 함께 사용하기
- 로그인된 사용자 정보 같이 다양한 컴포넌트에서 필요한 데이터는 context 사용하면 편함
[context + 비동기 API 연동 정석](https://codesandbox.io/p/sandbox/api-integrate-gtpcb?file=%2Fsrc%2FUsersContext.js%3A121%2C4)
[위의 코드 리팩토링]()(https://codesandbox.io/s/api-integrate-c3rli?fontsize=14)
# Fake API 서버 만들기
1. 리액트 프로젝트 디렉터리(src밖)에 data.json 파일 작성
	- data.json에 넣는 데이터들은 json객체들이어야하며, 각 객체의 key값으로 api요청할 수 있음
```json
// .../posts 로 요청
{
  "posts": [
    {
      "id": 1,
      "title": "리덕스 미들웨어를 배워봅시다",
      "body": "리덕스 미들웨어를 직접 만들어보면 이해하기 쉽죠."
    },
    {
      "id": 2,
      "title": "redux-thunk를 사용해봅시다",
      "body": "redux-thunk를 사용해서 비동기 작업을 처리해봅시다!"
    },
    {
      "id": 3,
      "title": "redux-saga도 사용해봅시다",
      "body": "나중엔 redux-saga를 사용해서 비동기 작업을 처리하는 방법도 배워볼 거예요."
    }
  ]
}
```
2. 터미널에서 서버 열기(처음 실행하면 json-server 자동으로 설치함)
```
npx json-server ./data.json --port 4000
```
3. 터미널에 뜨는 로그 보고 API 사용하기
# CORS와 웹팩 개발서버 Proxy
- 백엔드에서 프론트 도메인을 허용하기 위해 CORS 허용할 필요없이 웹팩 개발서버에서 제공하는 Proxy 기능 이용하면 됨
- CRA로 만든 리액트 프로젝트에서는 package.json에 `"proxy"`값만 설정하면 됨
```json
  ...
  },
  "proxy": "http://localhost:4000"
}
```
--> API 요청시 요청하는 주소의 도메인 생략 가능 --> 현재 페이지의 도메인을 가리키게됨
e.g. http://localhost:4000/posts --> /posts 로 요청

- 리액트로 만든 서비스의 도메인과 API의 도메인이 다르다면 axios의 글로번 baseURL설정하기
- 개발환경에서는 프록시 서버쪽으로 요청하고, 프로덕션에서는 실제 API서버로 요청하면 됨
```js
//index.js
axios.defaults.baseURL = process.env.NODE_ENV === 'development' ? '/' : 'https://api.velog.io/';
```