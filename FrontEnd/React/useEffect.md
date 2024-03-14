# useEffect

```table-of-contents
```

##  useEffect

- useEffect는 mount&update시에 html 렌더링 이후에 작동
- 오래걸리는 작업의 경우 return문 이전에 시간을 잡아먹는 대신 useEffect 안에 넣어서, 일단 렌더링 후에 작동하도록 시점을 조절 가능
- 즉 html 렌더링 외의 잡다한 기능을 useEffect에 넣기 가능
```jsx
import {useState, useEffect} from 'react';

function Detail(){

  useEffect(()=>{
    //여기적은 코드는 컴포넌트 로드 & 업데이트 마다 실행됨
    console.log('안녕')
  });

  let [count, setCount] = useState(0)

  return (
    <button onClick={()=>{ setCount(count+1) }}>버튼</button>
  )
}
```

## useEffect의 의존성 배열
- useEffect의 두번째 파라미터로는 `[]` 가능
- `[]` 안에 state나 변수 등을 넣을 수 있음
- 이 경우 `[]`안의 state나 **변수가 변할 때에만** useEffect 안의 코드를 실행해줌
	- `useEffect(()=>{ 실행할코드 }, [count])`
	- 배열 안에 여러 값이 있는 경우, 그 중 하나만 변경되어도 실행됨
- 아무것도 넣지 않은 경우에는 **mount시에만 1회만** 실행됨
	- `useEffect(()=>{ 실행할코드 }, [])`
- 의존성 배열을 전혀 넣지 않은 경우에는 **컴포넌트가 렌더링될 때마다 실행됨**



## Clean Up Function

- useEffect가 동작하기 전에 특정 코드를 실행하고 싶다면 `return () => {}`안에 넣어서 동작 가능

```jsx
useEffect(()=>{
  그 다음 실행됨
  return ()=>{
    여기있는게 먼저실행됨
  }
}, [count])
```

- 이러한 clean up function 기능은 타이머 제거, socket 연결 요청 제가, ajax 요청 중단 등에 쓰임
- 또한 컴포넌트 unmount 시에도 clean up function이 1회 실행됨


## async 와 useEffect

- axios나 setTimeout 등의 async 지원 함수들은 처리 순서가 꼬일 수 있음
- [useState](useState.md) 역시 처리 순서에 민감할 수 있음
- 따라서 이런 경우에는 useEffect를 통해서 순서를 지정하는 것이 좋음
```jsx
useEffect(()=>{
  if ( count != 0 && count < 3 ) { 
    setAge(age+1)
  }
 }, [count])
}
<button onClick={()=>{

  setCount(count+1);

}}>누르면한살먹기</button>
```
- count가 변경되면 변경된 뒤에 age가 변경되도록 구성

## (참고) class 형 컴포넌트에서만 사용 가능한 생애주기 제어

- ComponentDidMount
	- 탄생, Mount, 화면에 나타날 때
- ComponentDidUpdate
	- 변화, Update, 리렌더링 될 때
- ComponentWillUnmount
	- 죽음, Unmount, 화면에서 사라질 때