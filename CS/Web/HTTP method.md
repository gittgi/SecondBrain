# HTTP method

```table-of-contents
```


## GET

- [메세지 바디](../../미완성%20문서/HTTP%20body.md) 없이 URL의 쿼리 파라미터에 데이터를 포함시켜 전달
	- `http://localhost:8080/member?username=gittgi&rank=3`
		- URL 뒤에 ?로 시작해서 파라미터를 키=밸류 형식으로 넣고 &로 연결
	- 검색, 필터, 페이징 등에서 많이 사용하는 방식
	- URL에 붙이기 때문에 [HTTP header](../../미완성%20문서/HTTP%20header.md) 에 포함되어 전송됨
	- GET 방식에서는 Body가 빈 상태로 보내지기 때문에 Body 데이터를 설명하는 `Content-Type` 헤더 필드는 들어가지 않음
	- URL 형태로 표현되기 때문에 특정 페이지를 다른사람에게 접속하게 할 수 있음
	- 간단한 데이터를 넣도록 설계되었기 때문에, 보내는 데이터 양이 한정적임


## POST

- GET 방식과는 달리 데이터 전송을 목적으로 하는 요청 메서드
- [Body](../../미완성%20문서/HTTP%20body.md)에 데이터를 넣어서 보냄
	- 따라서 데이터를 설명하는 `Content-Type` 헤더 필드에 어떤 데이터 타입인지 명시
		- 대표적인 컨텐츠 타입
			- `application/x-www-form-urlencoded` : form으로 전송할 때 쓰는 타입, 모든 문자를 서버로 보내기전 인코딩; 따로 설정하지 않을 때의 기본값
			- `application/json` : json 형태
			- `text/plain` : 단순 txt
			- `multipart/form-data` : form이 파일이나 이미지를 전송할 때 (모든 문자를 인코딩하지 않음 -> 바이너리 데이터라는 것을 표시)
	- HTML Form 방식
		- `content-type: application/x-www-form-urlencoded`
		- 메세지 바디에 쿼리 파라미터 방식으로 전달 `username=gittgi&rank=3`
			- 회원 가입, 상품 주문, HTML Form 사용
	- HTTP message body에 데이터를 직접 담아서 요청
		- HTTP API에서 주로 사용하는 방식
		- 데이터 형식은 주로 [JSON](../JSON.md) 사용
		- POST 메서드 뿐만아니라 PUT, PATCH에서도 사용

