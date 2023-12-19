- 상태 변경으로 UI 반응하기: 리액트는 코드에서 직접 UI를 수정하지 않고 컴포넌트의 다양한 시각적 상태에 대해 보고자하는 UI를 설명하고, 사용자 입력에 대한 상태 변경을 트리거함
- 상태 구조 선택하기: 중복된 상태 제거하기, [[상호작용 추가하기]]의 상태 구조 원칙 참고
- 컴포넌트간 상태 공유(상태 끌어올리기)
- 상태 보존과 재설정: 리액트는 이전에 렌더링된 컴포넌트 트리와 일치하는 트리 부분을 보존. 기본 동작을 재정의하고 컴포넌트가 상태를 재설정해 새로운 컴포넌트를 렌더링하도록 강제할 수 있음
- 상태 로직을 Reducer로 추출하기: 많은 이벤트 핸들러에 걸쳐 많은 상태 업데이트가 분산되는 컴포넌트는 복잡해질 수 있음. 이럴때 컴포넌트 외부에 모든 상태 업데이트 로직을 단일 함수인 Reducer로 통합하고, 이벤트 핸들러는 사용자 '액션'만 지정해 Reducer함수는 파일의 맨 아래에서 각 액션에 대한 상태 업데이트 방식 지정
- Context를 사용하여 데이터 깊게 전달하기: 많은 컴포넌트로 일부 prop를 전달하거나 많은 컴포넌트가 동일한 prop가 필요할때, context를 이용하면 부모 컴포넌트가 하위 트리의 모든 컴포넌트에서 사용할 수 있도록 일부 정보를 명시적으로 전달하지 않고도 해당 정보를 사용할 수 있음
- Reducer와 Context를 사용하여 확장하기: Reducer로 컴포넌트의 상태 업데이트 로직 통합하고 context로 다른 컴포넌트로 깊이 있는 정보 전달 --> 복잡한 상태를 가진 상위 컴포넌트가 Reducer로 상태를 관리하고 트리의 깊은 곳에 있는 다른 컴포넌트는 context를 통해 해당 상태를 읽을 수 있음. 또한 상태를 업데이트하기 위해 액션을 디스패치할 수도 있음
# 사용자 입력 state 처리
---
https://reactnext-central.xyz/docs/react/managing-state/reacting-to-input-with-state

# 컴포넌트 간 state 공유(상태 끌어올리기)
---
- 컴포넌트 간 상태를 공유하려면 개별 컴포넌트에 상태를 두지 말고 모두를 포함하는 상위의 가장 가까운 컴포넌트에 상태를 두어 하위 컴포넌트에 props로 전달하면 됨
```jsx
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