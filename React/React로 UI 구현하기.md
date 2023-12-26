JSON API와 디자인 목업이 있다고 가정
# 1. 컴포넌트 계층으로 UI 분할하기
- 모든 컴포넌트와 하위 컴포넌트 주위에 상자를 그리고 이름 지정하기
- 하나의 컴포넌트는 한 가지 일만 해야함(단일 책임 원칙)
- UI를 컴포넌트로 분리
- 각 컴포넌트가 데이터 모델의 한 부분과 일치하도록 분리
- 컴포넌트 내에 나타나는 컴포넌트는 자식 컴포넌트가 됨

# 2. React로 정적 버전 구축하기
- 상호작용(state)을 추가하지 않고 데이터 모델로부터 UI를 렌더링한 정적 버전 구축하기
- 단순한 프로젝트는 상향식(상위 컴포넌트부터), 큰 프로젝트는 하향식 작업(하위 컴포넌트부터)이 쉬움
- 단방향 데이터 흐름으로 구축: 최상단 컴포넌트는 데이터 모델(JSON)을 props로 받음

# 3. UI 상태의 최소한의 완전한 표현 찾기
- UI들 중 최소한의 확실한 state를 찾기
- 시간이 지나도 값이 변경되지 않거나, 부모로부터 props를 통해 전달받거나, 컴포넌트 내에서 기존 상태나 props를 기반으로 계산할 수 있다면 상태가 아님

# 4. 상태(state)가 어느 컴포넌트에 있어야할지 식별하기
- 리액트는 부모 컴포넌트에서 자식 컴포넌트로 데이터를 내려보내는 단방향 데이터 흐름 사용
- 애플리케이션의 각 상태들에대해 다음 단계 수행:
1. 해당 상태를 기반으로 렌더링하는 모든 컴포넌트 식별
2. 그 중 최상위 컴포넌트 찾기
3. 상태가 위치할 곳 결정
	-  일반적으로 공통 부모 컴포넌트에 넣기
	- 공통 부모 컴포넌트 위의 컴포넌트에 넣기
	-  상태를 소유할만한 컴포넌트가 없다면 상태를 유지하기 위한 새로운 컴포넌트를 만들고 공통 부모 컴포넌트 위에 추가하기
	- 항상 함께 표시되는 상태들은 같은 위치에 넣기

# 5. 역 데이터 흐름 추가
- 상태를 전달할 컴포넌트의 onChange 이벤트 핸들러로 setSomething을 전달
```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```
- 상태를 전달받은 컴포넌트는 onChange이벤트 핸들러를 추가하고 부모의 상태 업데이트
```js
function SearchBar({
	filterText,
	inStockOnly,
	onFilterTextChange,
	onInStockOnlyChange
}) {
	return (
		<form>
			<input
				type="text"
				value={filterText} placeholder="Search..."
				onChange={(e) => onFilterTextChange(e.target.value)} />
			<label>
				<input
					type="checkbox"
					checked={inStockOnly}
					onChange={(e) => onInStockOnlyChange(e.target.checked)} />
					{' '}
					Only show products in stock
			</label>
		</form>
	);
}
```

