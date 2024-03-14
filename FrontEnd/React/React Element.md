---
aliases:
  - Element
---
# React Element

```table-of-contents
```

##  React Element

- 엘리먼트는 React 앱의 가장 작은 단위
- 앨리먼트는 화면에 표시할 내용을 기술
```jsx
const element = <h1>Hello, world</h1>;
```
- 브라우저의 DOM 엘리먼트와 달리, React 엘리먼트는 일반 객체
- 따라서 쉽게 생성할 수 있고, React DOM은 React 엘리먼트와 일치하도록 DOM을 업데이트


### DOM에 엘리먼트 렌더링

- React로 구현된 어플리케이션은 일반적으로 **하나의 루트 DOM 노드** 존재[^1]
- 이 안에 들어가는 모든 엘리먼트를 React DOM에서 관리
- React 엘리먼트를 렌더링 하기 위해서는 우선 DOM 엘리먼트를 `ReactDOM.createRoot()`에 전달
- 이후 React 엘리먼트를 `root.render()`에 전달
```html
<div id ="root"></div>
```
```jsx
const root = ReactDOM.createRoot(
  document.getElementById('root')
);
const element = <h1>Hello, world</h1>;
root.render(element);
```


### 렌더링 된 엘리먼트 업데이트하기

- React 엘리먼트는 불변객체(const) -> 엘리먼트를 생성한 이후에는 해당 엘리먼트의 자식이나 속성을 변경할 수 없음
- 따라서 엘리먼트는 마치 영화에서의 하나의 프레임처럼, 특정 시점의 UI를 보여주는 것
- 그러나 React DOM은 해당 엘리먼트와 그 자식 엘리먼트를 이전 엘리먼트와 비교하고, DOM을 원하는 상태로 만드는데 필요한 경우에만 DOM을 업데이트
- 이를 통해서, 전체 UI를 다시 그리도록 엘리먼트를 작성해도 React DOM은 내용이 변경된 부분만 업데이트
```js
const root = ReactDOM.createRoot(document.getElementById('root'));
  
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  root.render(element);
}

setInterval(tick, 1000);
```

- React DOM이 어떻게 부분만 렌더링 할 수 있는지는 [Virtual DOM](../DOM.md) 내용 참조


[^1]: React를 기존 앱에 통합하려는 경우에는 원하는 만큼 많은 수의 독립된 루트 DOM 노드가 있을 수 있음