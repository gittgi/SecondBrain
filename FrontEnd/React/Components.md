# Components

```table-of-contents
```

##  Components

- 컴포넌트를 통해 UI를 재사용 가능한 개별적인 여러 조각으로 나누는 것으로, 각 조각을 개별적으로 살펴 볼 수 있음


## 함수형 컴포넌트

```jsx
function Welcome(props) {
	return <h1>Hello, {props.name}</h1>;
}
```
- 데이터를 가진 하나의 props 객체 인자를 받아 [React Element](React%20Element.md)를 반환하는 유효한 React Component
- 컴포넌트가 JS 함수이기 때문에 **함수형 컴포넌트**라고 부름


>[!Warning] 컴포넌트 이름은 항상 대문자로 시작
>- React는 **소문자로 시작하는 컴포넌트는 DOM 태그**로 처리
>- 따라서 사용자 정의 컴포넌트는 꼭 대문자로 시작하도록 명명


## 클래스형 컴포넌트
```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
- ES6 class를 사용하여 컴포넌트를 정의 했기 때문에 **클래스형 컴포넌트** 라고 부름


## 함수형 컴포넌트 vs 클래스형 컴포넌트

- 함수형 컴포넌트가 클래스형 컴포넌트에 비해 더 가벼움
- 그러나 과거 버전 기준, 클래스형 컴포넌트에서만 생명 주기나 state 등을 활용할 수 있었음
- 버전이 업그레이드 됨에 따라 [useEffect](useEffect.md), [useState](useState.md) 등의 React Hooks의 도입으로 인해 함수형 컴포넌트에서도 생명 주기에 따른 작동이나 state를 활용할 수 있게 됨
- 따라서 현재는 가벼운 함수형 컴포넌트가 지배적으로 사용되고 있음



## 컴포넌트 렌더링

- 기존에는`<div>` 등의 DOM 태그만을 사용해 [React Element](React%20Element.md)를 나타냄
- 이제는 사용자 정의 컴포넌트로도 React Element를 나타낼 수 있음
```jsx
const element = <Welcome name="Sara" />;
```

- React는 사용자 정의 컴포넌트로 작성한 엘리먼트를 발견하면, JSX 어트리뷰트와 자식을 해당 컨포넌트에 단일 객체로 전달 -> 이것을 **props**라고 부름

- 컴포넌트 렌더링 과정
```jsx
function Welcome(props) {  
	return <h1>Hello, {props.name}</h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
const element = <Welcome name="Sara" />;
root.render(element);
```
1. `<Welcome name="Sara" />` 엘리먼트로 `root.render()`를 호출
2. React는 `{name: 'Sara'}`를 props로 하여 `Welcome` 컴포넌트를 호출
3. `Welcome` 컴포넌트는 결과적으로 `<h1>Hello, Sara</h1>` 엘리먼트를 반환
4. React DOM은 `<h1>Hello, Sara</h1>` 엘리먼트와 일치하도록 DOM을 효율적으로 업데이트


## 컴포넌트 추출

- 재사용 가능성 및 유지 보수, 가독성 등의 측면에서 컴포넌트를 여러 별도의 컴포넌트로 추출하여 구성하는 것이 유리할 수 있음

- 원래 Comment 컴포넌트
```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

- Avatar 컴포넌트 추출
```jsx
function Avatar(props) {
  return (
    <img className="Avatar"      src={props.user.avatarUrl}      alt={props.user.name}    />  );
}
```

- UserInfo 컴포넌트 추출
```jsx
function UserInfo(props) {
  return (
    <div className="UserInfo">      <Avatar user={props.user} />      <div className="UserInfo-name">        {props.user.name}      </div>    </div>  );
}
```

- 추출 이후 단순화된 Comment 컴포넌트
```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />      
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```



## props

- props 객체 인자는 **읽기 전용**
- 함수형 컴포넌트나 클래스형 컴포넌트 모두 컴포넌트 자체 props를 수정하면 안됨

>[!Warning]
>- 모든 React 컴포넌트는 자신의 props를 다룰 때 반드시 **순수 함수**처럼 동작해야 함


### 순수 함수

- 입력값을 바꾸려 하지 않아야 함
- 항상 동일한 입력 값에 대해 동일한 결과를 반환

#### 순수 함수 예제
```js
function sum(a, b) {
  return a + b;
}
```
#### 순수 함수가 아닌 예제
```js
function withdraw(account, amount) {
  account.total -= amount;
}
```
- 자신의 입력값인 account 객체의 값을 바꾸려고 하기 때문에 순수 함수가 아님