# Closure

```table-of-contents
```

##  Closure

- 함수가 속한 Lexical Scope를 기억하여 함수가 Lexical Scope 밖에서 실행될 때에도 이 scope에 접근할 수 있게 해주는 기능
- 함수 안에서 선언된 변수(a)를 함수 안에 선언된 또 다른 함수의 내부에서 부모 scope에 있는 변수(a)에 접근 가능
- 내부 함수에서 외부 함수(부모 함수)의 스코프의 변수에 접근할 수 있는데, 외부 함수의 함수 호출이 끝나고 return이 되었을 때에도 가능
- 이미 실행 컨텍스트의 큐에서 부모 함수의 컨텍스트 정보는 모두 사라졌지만 자식 함수가 남아있다면, **그 자식 함수에서 이미 종료된 부모 함수의 컨텍스트 정보(변수나 함수 등의 정보)를 참조**할 수 있다.

### 예시
```js
function func1(){
	let variable = 0;

	return function(){
		variable += 1;
		return variable
	}
}
const add = func1();
console.log(add());
```
- const add = func1(); 에서 func1()는 실행 완료
- 그러나 add에 남아있는 func1의 내부 익명함수가 부모함수인 func1의 변수 정보를 가지고 있기 때문에, add의 익명함수를 호출할 때, 해당 변수의 정보를 업데이트 하고 return 가능
