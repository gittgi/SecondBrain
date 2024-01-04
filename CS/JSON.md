---
tags:
  - CS_지식의_정석
---

# JSON

```table-of-contents
```

##  JSON이란?

- JSON(JavaScript Object Notation)은 JavaScript 객체 문법으로 구조화된 데이터 교환 형식
- javascript, python, java 등 여러 언어에서 데이터 교환형식으로 활용
- 객체 문법 뿐만 아니라  배열, 문자열도 표현 가능
```json
{
	"name" : "junhyung",
	"id" : "gittgi"
}
```

## JSON의 특징

### 1. Javascript 문법
- 키(key)와 값(value)로 구성
- 이미 존재하는 키를 중복으로 선언하면, 나중에 선언한 해당 키에 대응하는 값으로 덮어 쓰여짐

### 2. 독립적인 형식
- 특정 언어에 종속되지 않고 독립적이기 때문에, 다양한 언어에서 객체, 해시테이블, 딕셔너리 등으로 변환되어 사용될 수 있다
- 따라서 서로 다른 언어의 시스템 사이에서 데이터를 교환하는데 널리 사용됨

### 3. 단순 배열, 문자열 표현
- 단순 배열 : `[1, 2, 3, 4]`
- 문자열 : `"gittgi"`
- 꼭 key - value 형태가 아닌 배열이나 문자열도 가능

## JSON의 타입

- JS의 object와 유사하지만, undefined와 메서드 등은 포함할 수 없다.
- JSON이 포함할 수 있는 타입
	- 수 (Number)
	- 문자열 (String)
	- 불린 (Boolean)
	- 배열 (Array)
	- 객체 (Object)
	- null


## JSON의 직렬화와 역직렬화

- 외부의 시스템에서도 사용할 수 있도록 바이트(byte) 형태로 데이터를 변환하는 기술이 **직렬화**
- 반대로 해당 시스템에 맞는 형태로 변환하는 기술을 **역직렬화**
- 자바스크립트의 `JSON.parse()`(역직렬화), `JSON.stringify()`(직렬화)

## JSON의 활용
- JSON은 프로그래밍 언어와 프레임워크에 독립적
- 따라서 서로 다른 시스템 간에 데이터 교환에 적합
- 대표적으로 API 반환 형태나, 시스템 설정파일(package.json) 등에 주로 활용됨
