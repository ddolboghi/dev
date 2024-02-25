# 리덕스 사용하는 이유
- 리덕스는 중앙 집중식 상태 관리 체계임
- 따라서 같은 상태를 실시간으로 공유하는 컴포넌트들이 있을때, props waterfall 없이 `useSelector`등을 이용하여 동시에 같은 값을 공유할 수 있음
# 리덕스에서 사용되는 키워드
## action
- 상태에 어떤 변화가 필요할때 액션을 발생시킴
- 액션은 하나의 객체로 표현됨
- 액션 객체는 `type`필드를 필수적으로 가져야하고, 그 외의 값들은 선택
	- payload: type이나 status를 제외한 액션의 모든 정보 가짐
	- error: 액션에 에러 발생시 true가 되고, 이때 payload는 error객체가 되야함
	- meta: payload의 일부가 될 수 없는 추가적인 정보
```js
{
  type: "TOGGLE_VALUE"
  data: {
	id: 0,
	text: "리덕스 배우기"
  }
}
```
## 액션 생성함수
- 액션을 만드는 함수로, 파라미터를 받아 액션 객체를 반환
- 컴포넌트에서 더 쉽게 액션을 발생시키기 위해 만듬
- export로 다른 파일에서 불러와 컴포넌트에서 사용
- 리덕스에서 필수는 아님, 액션 발생시킬때마다 직접 액션 객체 작성 가능
```js
export function addTodo(data) {
  return {
    type: "ADD_TODO",
    data
  };
}

// 화살표 함수로도 만들 수 있습니다.
export const changeInput = text => ({ 
  type: "CHANGE_INPUT",
  text
});
```
## Reducer
- 변화를 일으키는 함수로, **현재 상태값**과 **액션**을 전달받아 **상태 업데이트** 후 반환
- 카운터를 위한 리듀서는 switch문 사용
	- **defalut문에서는** 에러 발생시키지 말고 **전달받은 상태값을 그대로 반환**해야함
- 리덕스 사용할때 여러 리듀서를 만들고 이를 합쳐 rootReducer를 만들 수 있음
```js
function counter(state, action) {
  switch (action.type) {
    case 'INCREASE':
      return state + 1;
    case 'DECREASE':
      return state - 1;
    default:
      return state; //기
  }
}
```
## Store
- 리덕스에는 한 애플리케이션당 하나의 store를 만듬
- store안에는 현재 앱의 state와 리듀서가 있음
- store는 내장함수도 있음
## dispatch
- 액션을 발생시키는 함수로, store의 내장함수 중 하나
- dispatch함수에 액션을 파라미터로 전달 e.g. `dispatch(action)`
1. dispatch함수를 호출하면 store가 reducer함수를 실행시킴
2. dispatch함수에 전달한 액션을 처리하는 로직이 reducer함수에 있다면 새로운 상태를 만듬
## subscribe
- 함수를 파라미터로 받아 액션이 dispatch되었을때마다 전달받은 함수를 호출하는 store 내장 함수
- 리덕스 사용시 subscribe함수를 직접 사용하는 일은 별로 없고, react-redux라이브러리에서 제공하는 `connect`함수나 **`useSelector`훅으로 리덕스 store의 상태에 구독함**
# 리덕스의 3가지 규칙
## 1. 하나의 애플리케이션 안에는 하나의 store만 있다.
- 하나의 애플리케이션에서 여러개의 store를 사용하는 건 권장하지 않음
- 특정 업데이트가 빈번하게 일어나거나 애플리케이션의 특정 부분을 완전히 분리시키게 될때 여러개의 store를 만들수도 있지만 개발도구를 활용하지 못함
## 2. 상태(state)는 읽기 전용이다.(불변성 유지)
- 객체나 배열의 원본을 수정하지 않고 복사본을 수정하는 것 처럼 상태값도 원본을 업데이트하지 않고 복사한 상태값을 업데이트함
- 리덕스는 shollow equality 검사를 해서 객체의 변화를 감지할때 겉만 비교해 좋은 성능 유지함
- Immutable.js, Immer.js로 불변성 유지하며 상태관리할 수 있음
## 3. Reducer함수는 순수 함수여야 한다.
- reducer함수는 이전 state와 action 객체를 파라미터로 받음
- 이전 state는 절대 건들이지 않고 변화를 일으킨 새로운 state를 반환
- 똑같은 파라미터로 호출된 reducer함수는 항상 똑같은 결과값을 반환해야함
	- `new Date()`, 랜덤 숫자 생성, 네트워크 요청 같은 로직은 실행할때마다 결과값이 변하므로 **리덕스 미들웨어**를 사용해 reducer함수 밖에서 처리해야함
# 리덕스 설치
```cmd
yarn add redux
npm install @reduxjs/toolkit //리덕스를 간편하게 사용하게 해주는 라이브러리
yarn add react-redux //리덕스를 리액트에 적용하기 위한 라이브러리
```
[redux-toolkit v2.0.0 릴리즈 노트](https://github.com/reduxjs/redux-toolkit/discussions/3945)

> [!info] Ducks 패턴
> 액션 타입, 액션 생성함수, 리듀서를 한 파일에 작성하는 패턴
[공식 깃헙](https://github.com/erikras/ducks-modular-redux)
# 리덕스 모듈 만들기
- 리덕스 모듈: 액션 타입, 액션 생성함수, 리듀서가 들어있는 js 파일
- **modules** 디렉터리해 생성해주기
- redux toolkit의 `createSlice`, `configureStore` 사용
- redux toolkit은 immutable update를 위해 내부적으로 immer.js 사용함
## `combineReducers` from "redux"
- 리듀서들을 모아서 rootReducer를 만들때 사용
```js
import { combineReducers } from "redux";
import counterReducer from "./counter";
import todosReducer from "./todos";

const rootReducer = combineReducers({
  counter: counterReducer,
  todos: todosReducer,
});

export default rootReducer;
```
## `configureStore` from "@reduxjs/toolkit"
- 새로운 store만드는 함수
```js
export const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger),
  devTools: process.env.NODE_ENV !== 'production',
  preloadedState: initialState,
  enhancers: (defaultEnhancers) => [...defaultEnhancers]
});
```
- reducer: 단일 함수로 설정하면 store의 rootReducer로 사용됨(`combineReducers`로 slice reducer들을 병합한 rootReducer 포함), slice reducer로 설정하면 자동으로 `combineReducers`에 전달해 rootReducer 생성
- middleware: 미들웨어를 설정하면 자동으로 applyMiddleware에 전달, 미들웨어 설정 안하면 getDefaultMiddleware 호출, **미들웨어를 배열에 담아 콜백 형태로 작성해야함**( 😱따로 배열 만들어 콜백하면 기본 미들웨어가 무시됨!!)
- devTools: Redux DevTools 사용 여부, 기본값 true
- preloadedState: redux store의 초기값(initialState)
- enhancers: 사용자 정의 미들웨어, 콜백 함수로 설정 시 미들웨어 적용 순서 정의 가능
## `createSlice` from @reduxjs/toolkit
- 선언한 slice reducer의 name에 따라 액션 생성자, 액션 타입, 리듀서를 자동 생성해줌(`createAction`, `createReducer`가 내장됨)
- 액션 타입명은 '**{name}/{slice명}**'으로 생성됨
	- 아래 코드에서는 'counter/setDiff', 'counter/increase', 'counter/decrease'가 됨
- 개별 slice reducer를 외부 dispatch함수에서 사용하려면 반드시 `mySlice.actions`를 export해야함
됨
### 파라미터
- name: 해당 모듈의 이름
- initialState: 해당 모듈의 초기값
- reducers: **state**와 **action**을 파라미터로 받는 리듀서 정의, 해당 리듀서의 키값(함수명)으로 액션함수가 자동으로 생성됨
- extraReducers: 이미 다른 곳에서 정의된 액션 생성함수를 지정
	- 해당 slice의 initialState 값을 외부에서 정의된 액션 생성함수로 변경하고 싶을때 사용
	- 외부 액션 생성함수를 지정하므로, 새로운 액션 타입을 생성하지 않음
	- 주로 redux-thunk의 `createAsyncThunk`로 정의된 액션 생성함수를 사용하거나 다른 slice에서 정의된 액션 생성함수를 사용하는 경우 이들을 extraReducers에 추가함
	- 빌더 패턴으로 switch문과 유사하게 작성해 프로미스의 진행 상태에 따라 개별적으로 리듀서를 실행할 수 있음
```js
import { createSlice } from "@reduxjs/toolkit";
  
const initialState = {
  number: 0,
  diff: 1,
};

const counterSlice = createSlice({
  name: "counter",
  initialState,
  reducers: {
    setDiff(state, action) {
      state.diff = action.payload.data;
    },
    increase(state, action) {
      state.number = state.number + state.diff;
    },
    decrease(state, action) {
      state.number = state.number - state.diff;
    },
  },
});

export const { setDiff, increase, decrease } = counterSlice.actions;
export default counterSlice.reducer;
```

```js
import { createSlice } from "@reduxjs/toolkit";
  
let nextId = 1;
  
const initialState = [
  /*{ id: 1, text: '예시', done: false } */
];
  
const todoSlice = createSlice({
  name: "todo",
  initialState,
  reducers: {
    addTodo(state, action) {
      const newTodo = {
        id: nextId++,
        text: action.payload,
        done: false,
      };
      state.push(newTodo);
    },
    toggleTodo(state, action) {
      state.map((todo) =>
       todo.id === action.payload ? { ...todo, done: !todo.done } : todo
      ); //불변성 지키기 위해 새로운 객체 생성
    },
  },
});
  
export const { addTodo, toggleTodo } = todoSlice.actions;
export default todoSlice.reducer;
```

기존 redux 코드
```js
const SET_DIFF = 'counter/SET_DIFF';
const INCREASE = 'counter/INCREASE';
const DECREASE = 'counter/DECREASE';

//액션 생성함수
export const setDiff = diff => ({ type: SET_DIFF, diff });
export const increase = () => ({ type: INCREASE });
export const decrease = () => ({ type: DECREASE });

const initialState = {
  number: 0,
  diff: 1
};

//리듀서 선언
export default function counter(state = initialState, action) {
  switch (action.type) {
    case SET_DIFF:
      return {
        ...state,
        diff: action.diff
      };
    case INCREASE:
      return {
        ...state,
        number: state.number + state.diff
      };
    case DECREASE:
      return {
        ...state,
        number: state.number - state.diff
      };
    default:
      return state;
  }
}
```
# 모듈 사용하기
## `useDispatch` from 'react-redux'
- 리덕스 store의 dispatch를 함수에서 사용할 수 있게 해주는 훅
- 각 액션을 디스패지하는 함수들을 만들어야함
```js
//...
import { useSelector, useDispatch } from 'react-redux';
import { setDiff, increase, decrease } from "../modules/counter";
  
function CounterContainer() {
  const number = useSelector((state) => state.counter.number);
  const diff = useSelector((state) => state.counter.diff);
  
  const dispatch = useDispatch();
  const onIncrease = () => dispatch(increase());
  const onDecrease = () => dispatch(decrease());
  const onSetDiff = (diff) => dispatch(setDiff(diff));
  
  return (
    <Counter
      number={number}
      diff={diff}
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onSetDiff={onSetDiff}
    />
  );
}
  
export default CounterContainer;
```
useCallback 기억이 안난다면 [[고급 기법#useCallback]]
## `useSelector` from "react-redux"
- 리덕스 store에서 state를 조회해 필요한 state만 선택해서 가져오는 함수
## 프로젝트에 리덕스 적용하기
- src디렉터리에 index.js에서 rootReducer를 불러와 `configureStore`로 새로운 `store`를 만들고 `<Provider store={store}>`를 사용해 프로젝트에 적용
```js
//...
import rootReducer from "./modules";
import { configureStore } from "@reduxjs/toolkit";
import { Provider } from "react-redux";
  
const store = configureStore({
  reducer: rootReducer,
});
  
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```
## container 컴포넌트에서 사용하기
- container 컴포넌트는 특정 리듀서에서 액션을 import해 dispatch하여 액션 함수를 만들고 이를 자식 컴포넌트에 props로 넘겨줌
```js
import React from "react";
import { useSelector, useDispatch } from "react-redux";
import Counter from "../components/Counter";
import { setDiff, increase, decrease } from "../modules/counter";
  
function CounterContainer() {
  const number = useSelector((state) => state.counter.number);
  const diff = useSelector((state) => state.counter.diff);
  
  const dispatch = useDispatch();
  const onIncrease = () => dispatch(increase());
  const onDecrease = () => dispatch(decrease());
  const onSetDiff = (diff) => dispatch(setDiff(diff));
  
  return (
    <Counter
      number={number}
      diff={diff}
      onIncrease={onIncrease}
      onDecrease={onDecrease}
      onSetDiff={onSetDiff}
    />
  );
}
  
export default CounterContainer;
```
## presentational 컴포넌트에서 props로 받기
- presentational 컴포넌트는 필요한 액션 함수들을 props로 받음
```js
import React from "react";
  
function Counter({ number, diff, onIncrease, onDecrease, onSetDiff }) {
  const onChange = (e) => {
    //e.target.value는 문자열 타입이므로 숫자로 변환
    onSetDiff(parseInt(e.target.value, 10));
  };
  return (
    <div>
      <h3>{number}</h3>
      <div>
       <input type="number" value={diff} min="1" onChange={onChange} />
        <button onClick={onIncrease}></button>
        <button onClick={onDecrease}></button>
      </div>
    </div>
  );
}
  
export default Counter;
```
# `useSelector` 최적화로 리렌더링 줄이기
- 리덕스 store의 상태를 조회하는 훅
- `useSelector`는 컴포넌트가 렌더링되거나 액션이 디스패치될때마다 selector함수를 실행하고 selector함수의 반환값을 반환함
- selector함수는 store의 state를 받음
## 리렌더링되는 경우
1. selector함수가 `state => state`일 경우, store 전체를 비교하기때문에 현재 컴포넌트와 상관 없는 state가 변경되면 리렌더링됨
2. selector함수의 반환값이 참조값이면 실제값은 그대론데 참조가 달라진 경우, 값이 바뀐 것으로 인식해 리렌더링됨(e.g. filter메서드로 반환되는 배열을 반환하는 경우)
## 최적화 방법
```js
//기존 코드
const { number, diff } = useSelector(state => ({
    number: state.counter.number,
    diff: state.counter.diff
}));
```
### 여러 개의 useSelctor 사용하기
- 독립적으로 선언된 상태들은 서로의 영향을 받지 않음
- 공식문서 추천 방법
```js
const number = useSelector((state) => state.counter.number);
const diff = useSelector((state) => state.counter.diff);
```
### shallowEqul 사용하기
- 기존 코드는 상태값이 변화와 무관하게 매번 새로운 객체를 반환해 매번 새로운 참조를 가져 리렌더링 발생
- `shallowEqual`은 객체의 겉 값만 비교하므로 리렌더링 방지
- **`useSelector`의 두번째 인자로 `shallowEqual` 넣어주기**
	- `useSelector`의 두번째 인자는 이전 값과 다음값을 비교해 false면 리렌더링함
```js
const { number, diff } = useSelector(state => ({
    number: state.counter.number,
    diff: state.counter.diff
  }),
  shallowEqual
);
```
### redux toolkit(RTK)의 createSelector 사용하기
[공식 문서](https://redux-toolkit.js.org/api/createSelector)
[사용방법 참고](https://velog.io/@domandjerry/createSelector-%ED%95%84%ED%84%B0-%EA%B8%B0%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94)
- `createSelector`함수는 메모이제이션을 활용해 인자로 받는 값이 변하지 않으면 메모이제이션된 결과값이 반환됨
- 기존의 `useSelector`는 컴포넌트가 상태를 처리해 컴포넌트가 리렌더링될 수 있지만 `createSelector`는 상태 변경 작업을 리덕스에서 미리 처리해 컴포넌트는 그냥 가져다 쓸 수 있음
- 컴포넌트에서 `useSelector`로 정의한 셀렉터 이름만 소환하면 됨
- _@param_ -` inputSelectors`: 첫번째 인자로 셀렉터들을 담은 **배열** 넣음
	- 배열로 셀렉터를 전달하는 이유는 캐시 관리와 두번째 인자의 함수가 어떤 상태에 의존하고 있는지 명시하기 위함
- _@param_ - `resultFunc`: 두번째 인자로, `inputSelectors`에 있는 셀렉터들의 결과를 인수로 받아 최종 결과를 계산하는 함수를 넣음
- _@param_ - `createSelectorOptions?`: 셀렉터마다 추가적인 커스텀을 허용하는 옵션 객체
```js
const selectTodosByCategory = createSelector(
  [
    // Pass input selectors with typed arguments
    (state: RootState) => state.todos,
    (state: RootState, category: string) => category
  ],
  // Extracted values are passed to the result function for recalculation
  (todos, category) => {
    return todos.filter(t => t.category === category)
  }
)
```

```js
export const selectAllPosts = (state) => state.posts.posts;

export const selectPostsByUser = createSelector(
  [
    selectAllPosts,
    (_state, userId) => userId
  ],
  (posts, userId) => posts.filter(post => post.user === userId)
);
```