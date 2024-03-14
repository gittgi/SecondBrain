# JSX

```table-of-contents
```

##  JSX

- JavaScript를 확장한 문법
- JSX는 React element를 생성
- Reactsms 별도의 파일에 마크업과 로직을 넣어 기술을 인위적으로 분리하는 대신, 둘 다 포함하는 "컴포넌트"라는 "느슨하게 연결된 유닛"으로 관심사를 분리

## JSX 특징

### 표현식 포함하기

- JSX의 중괄호 안에는 유효한 모든 JavaScript 표현식을 넣을 수 있음
```jsx
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!  </h1>
);


```

### 표현식으로 사용

- 컴파일이 끝나면, JSX 표현식이 정규 JavaScript 함수 호출 후에 JavaScript 객체로 인식됨
- 따라서 JSX를 if 문이나 for 문을 적용하거나, 변수에 할당, 인자로 받기, 함수로 부터 return 등이 가능해짐
```jsx
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;  }
  return <h1>Hello, Stranger.</h1>;}
```


### 속성 정의하기

- 속성에 따옴표를 이용해 문자열 리터럴 정의 가능
```jsx
const element = <a href="https://www.reactjs.org"> link </a>;
```

- 중괄호를 통해서 JS 표현식을 삽입하는 것도 가능
```jsx
const element = <img src={user.avatarUrl}></img>;
```

>[!Important]
>- JSX는 camelCase 프로퍼티 명명 규칙 사용
>- class : className, tabindex : tabIndex


### 자식 정의하기

- JSX 태그는 자식 포함 가능
```jsx
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### 주입 공격 방지

- React DOM은 기본적으로 JSX에 삽입된 모든 값을 렌더링하기 전에 이스케이프[^1]
- 따라서 어플리케이션에서 명시적으로 작성되지 않은 내용은 주입되지 않음
- 모든 항목은 렌더링 되기 전에 문자열로 변경되기 떄문에 [XSS](../../미완성%20문서/XSS.md) 공격을 방지할 수 있음


### 객체 표현 (React Element)

- [Babel](../../미완성%20문서/Babel.md)은 JSX를 `React.createElement()`호출로 컴파일
- 즉, 다음 두 코드는 동일
```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```js
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

- 이때 `React.createElement()`는 버그를 방지하기 위해 몇가지 검사를 수행한 후 다음과 같은 객체를 생성
```js
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```
- 이러한 객체가 [React Element](React%20Element.md)로, 화면에서 보고 싶은 것을 나타내는 표현
- React는 이 객체를 읽어서 DOM을 구성하고, 최신 상태로 유지하는데 사용



[^1]: 이스케이프 : HTML에서 이스케이프 한다는 것은, <, >, ", & 등 몇가지 특별한 문자들을 교체하는 것