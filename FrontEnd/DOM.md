---
aliases:
  - 가상 돔
  - Virtual DOM
---
# DOM

```table-of-contents
```

##  브라우저 렌더링

- 브라우저 : 웹에서 페이지를 찾아서 보여주고, 사용자가 하이퍼링크를 통해 다른 페이지로 이동할 수 있도록 하는 프로그램(MDN)
- 렌더링: 서버로부터 HTML, CSS, JavaScript 등 작성한 파일을 받아 브라우저에 뿌려주는 것

### 브라우저 렌더링 5단계

1. Parsing
	- HTML 파일과 CSS 파일을 서버로부터 받아와 파싱
	- 이를 바탕으로 DOM tree와 CSSOM tree를 생성
2. Style
	- 두 트리를 합쳐 Rendering Tree 생성
3. Layout/Reflow
	- Rendering Tree에서 각 노드의 위치와 크기를 계산
4. Painting(repaint)
	- 계산된 값을 이용해 각 노드를 화면상의 실제 픽셀로 변환, 레이어 생성
5. Composite
	- 레이어를 합성하영 실제 화면에 출력
	- 


## DOM

- Document Object Model, 문서 객체 모델
- html 파일을 웹 브라우저에서 보여주게 하기 위해서는 `브라우저 렌더링 엔진`은 웹 문서를 parsing하여 브라우저가 이해할 수 있는 구조로 구성하여 메모리에 올림 
	- 이 구조 모델이 바로 **DOM**
- 모든 요소, 텍스트, attribute들은 각각 객체로 만들어지고, 이 객체들은 위와 같이 트리 구조로 구성되어있기 때문에 DOM Tree라고 부름
- 다시 말하면, DOM tree는 각 노드들로 이루어져있고, 이 노드들은 각각 객체들
- 화면에 그려진 DOM은 DOM API를 통해 Javascript와 같은 스크립팅 언어로 동적으로 조작할 수 있고, 이 변경된 DOM은 rendering에 반영

>[!Information] DOM을 사용하는 API들의 목록
> - document.getElementById(id)
> - document.getElementsByTagName(name)
> - document.createElement(name)
> - parentNode.appendChild(node)
> - element.innerHTML 
> - element.setAttribute 
> - element.getAttribute
> - element.addEventListener

- DOM 조작 예시
```js
document.querySelector(‘#title”).style.color = “red”;
```


## Virtual DOM

### 가상 돔 등장 배경

- 화면에 보여지는 데이터 값이 수정되면, 브러우저의 렌더링 과정 전체가 다시 수행되는 것이 비효율
- SPA로 된 웹페이지들이 늘어나면서, 사용자 인터렉션에 의해 화면이 즉각적으로 수정되어야 하는 경우가 증가
- CSR 방식으로 렌더링 되는 경우에는, 처음 렌더링 때를 제외하고는 JS가 뷰를 관리하기 때문에, 브라우저에서 연산이 많아지면 렌더링 속도가 많이 저하될 수 있음
- 이러한 문제점들을 해결하기 위해 가상 돔이 등장


### 가상 돔

- Virtual DOM은 Real Dom을 컴포넌트 단위로 쪼개어 추상화시킨 메모리상에만 존재하는 복사본
- html객체에 기반한 JS객체로 표현되고, 실제 DOM이 아닌 메모리 상에서만 동작
```html
<ul id="items"> 
	<li>Item 1</li> 
	<li>Item 2</li>
</ul>
```
```js
let domNode = { 
	tagName: "ul", 
	attributes: { 
		id: "items", 
	}, 
	children: [ 
		{ 
			tagName: "li", 
			textContent: "Item 1", 
		}, 
		{ 
			tagName: "li", 
			textContent: "Item 2", 
		 }, 
	 ], 
 };
```

### 가상 돔 적용

- 가상 돔이 적용된 업데이트 과정
	- 데이터가 변경됨
	- 전체가 먼저 **가상 돔**에 렌더링
	- **이전** 가상 돔에 있던 내용과, 변경 된 내용을 비교
	- 바뀐 부분만 실제 돔에 적용
- 변경되는 부분만 실제 돔에 적용하기 때문에, 연산 비용 측면에서 효율적
- 수정 사항이 여러 개 있어도 돔 요소의 변화를 **묶어서** 적용 후, 이를 실제 돔에 동기화
	- 즉, 실제 돔에는 렌더링이 **1번**만 발생


### React에서의 Virtual DOM 렌더링

- React의 두개의 가상 돔을 사용
	1. **렌더링 이전 화면 구조**를 나타내는 가상돔
	2. **렌더링 이후에 보이게 될 화면 구조**를 나타내는 가상돔
- React에서 데이터 업데이트 시 가상 돔에서 실제 돔으로 렌더링 되는 과정
	1. 데이터 업데이트
	2. React.createElement()를 통해 [JSX Element](React/React%20Element.md)를 렌더링
	3. 모든 가상 돔 오브젝트 업데이트
	4. React는 가상 돔을 이전 가상 돔의 스냅샷과 비교하여 바뀐 부분을 캐치
- 바뀐 부분을 체크할 때 **Diffing Algorithm**을 통해 검사하게 됨
	- element의 속성 값만 바뀐 경우 -> **속성 값만 업데이트**
	- element의 태그 혹은 컴포넌트가 변경된 경우 -> **해당 노드를 포함한 하위 노드를 unmount 후 새로운 가상 돔으로 대체**
- 이렇게 Diffing 알고리즘을 통해 확인한 변경 부분들을 DOM object에 적용하는 것이 **재조정(reconciliation)**


### 가상 돔은 만능인가?

- 정보만 제공하는, 사용자의 인터렉션이 많지 않은 페이지의 경우에는 Real DOM이 성능이 더 좋을 수 있다.