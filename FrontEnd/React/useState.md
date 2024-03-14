---
aliases:
  - setState()
  - state
---
# useState

```table-of-contents
```

##  useState()

```jsx
// state와 setState의 이름은 변경하여 사용(ex. count, setCount)
const [state, setState] = useState<stateType>(initialValue);
```

- state에 저장하고 싶은 값을 저장
- `setState()`함수로 state 값을 변경
- 데이터는 동기적으로 업데이트 되지 않음 (비동기처리)
	- 페이지의 수많은 useState 데이터가 모두 동기적으로 업데이트 된다면, 리렌더링을 계속 발생시켜 성능 저하 가능성
- 따라서 useState는 setState가 연속적으로 호출되면 데이터를 배치(batch) 처리를 통해 한번에 처리 (16ms를 기준으로 모아서 한 번씩 업데이트)
- state의 값이 변동될 때마다 해당 컴포넌트를 **리렌더링**
	- 참고로 재렌더링이 이루어지는 4가지 조건
		- state 변경
		- 새로운 props
		- 기존 props 업데이트
		- 부모 컴포넌트가 재렌더링 될 때

### 사용 예시
```jsx
function App(){

  let [ 따봉, 따봉변경 ] = useState(0);
  return (
      <h4> { 글제목[0] } <span onClick={ ()=>{ 따봉변경(따봉 + 1) } } >👍</span> { 따봉 }</h4>
  )
}
```

- Array나 object 형인 state 변경은 spread 연산자를 이용해서 복사를 한 뒤에 변경하고 다시 저장
```jsx
function App(){

  let [글제목, 글제목변경] = useState( ['남자코트 추천', '강남 우동맛집', '파이썬 독학'] );

  return (
    <button onClick={ ()=>{
      let copy = [...글제목];
      copy[0] = '여자코트 추천';
      글제목변경(copy)
    } }> 수정버튼 </button>
  )
}
```

## useState와 Closure

- useState는 [Closure](../Closure.md)를 이용하여 함수의 상태를 기억

### 예시
```js
const simpleUseState = (initialValue) => {
	let innerState = initialValue;
	const state = () => innerState;
	const setState = (newValue) => {
		innerState = newValue;
	};
	return [state, setState];
};

// setState(10);
```
- simpleUseState를 실행하면 파라미터의 initialValue를 innerState에 할당
- state를 호출하면 이 innerState를 return한다
- setState를 호출하면 파라미터로 넘어온 newValue를 새로 innerState에 할당
- 실제로 state와 setState를 사용하는 시점은 simpleUseState 실행이 완료된 이후이지만, Closure가 innerState의 값을 기억하고 있기 때문에 그 이후에도 사용이 가능
- 해당 예시에서는 `const state = () => innerState;`형식으로 함수로 선언했지만 실제처럼 만들기 위해서는 변수로 할당하여야 함
- 그러나 const state = innerState;의 형식으로 바꾸게 되면 말 그대로 변수이기 때문에 setState함수를 실행하여도 값이 재할당 되지 않음
	- 따라서 state를 useState 메소드 외부에 배열 형식으로 저장

### React에서의 useState

- 리액트에서는 useState를 통해 생성한 상태를 접근하고 유지하기 위해서 useState 바깥에 state 를 저장
- 이 state들은 키로 접근할 수 있는 배열 형식으로 저장
- 따라서 useState를 통해 생성한 상태들은 배열에 순서대로 저장되며, 상태가 업데이트 되었을 때, 이 상태들은 컴포넌트 밖에서 선언되어 있는 변수들이기 때문에 업데이트 이후에도 계속 접근 가능


### 예시로 만들어진 간단한 useState
```js
import React from 'react';

let state = [];
let setter = [];
let cursor = 0;
let firstRun = true;

const createSetter = (cursor) => {  // Closure
	return (newValue) => {
		state[cursor] = newValue;
	}
}

const customUseState = (initialValue) => {
	if(firstRun) { // 첫번째 렌더링할 때, 
		state.push(initialValue);  // state에 initialValue 셋
		setter.push(createSe(cursor);  // useState 부를 때마다 값
		firstRun = false;
	}
	const resState = state[cursor];
	const resSetter = setter[cursor];
	cursor++;
	return [resState, resSetter];
};

export default function App () {
	cursor = 0;
	const [value, setValue] = customUseState(0);

	return (
		<div>
			<div>{ value }</div>
			<button onClick={() => setValue(value + 1)}>+</button>
			<button onClick={() => setValue(value - 1)}>-</button>
		</div>
	)
}
```
