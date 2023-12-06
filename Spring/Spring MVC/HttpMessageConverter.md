---
aliases:
  - 메세지컨버터
tags:
  - MVC
  - Spring
---
# HttpMessageConverter

```table-of-contents
```

##  HttpMessageConverter란?

- HTTP 요청과 응답에 대해, [HTTP body](../../미완성%20문서/HTTP%20body.md)에 들어가는 내용을 적합한 형식으로 변환해주는 객체
- [@RequestBody, @ResponseBody, HTTPEntity](Controller.md) 처럼 바디에 직접 연관되는 경우, [스프링 MVC](스프링%20MVC.md)는 메세지 컨버터를 적용
- HTTP 메세지 컨버터 인터페이스를 기반으로 구현된 여러 메세지컨버터들이 기본으로 등록되어 있음
	- 클래스타입과 [HTTP header](../../미완성%20문서/HTTP%20header.md)의 Content-Type, Accpet 헤더(정확히는 [@RequestMapping의 produces 속성](Controller.md)) 를 기준으로 메세지 컨버터 선정


## HTTP 메세지 컨버터 인터페이스

```java
// org.springframework.http.converter.HttpMessageConverter
    package org.springframework.http.converter;
    public interface HttpMessageConverter<T> {

      boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
      boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

      List<MediaType> getSupportedMediaTypes();

      T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
              throws IOException, HttpMessageNotReadableException;

      void write(T t, @Nullable MediaType contentType, HttpOutputMessage
    outputMessage)

              throws IOException, HttpMessageNotWritableException;

}
```

- 메세지 컨버터는 요청 / 응답 모두 대응
	- `canRead()` , `canWrite()` 메서드를 통해 지원하는 클래스, 미디어 타입인지 확인
	- `read()`, `write()` 메서드를 통해 메세지를 읽고 씀

## 기본 메세지 컨버터 구현체

```java
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter
2 = MappingJackson2HttpMessageConverter
.
.
.
```

- [스프링부트](../../미완성%20문서/SpringBoot.md)가 제공하는 다양한 메세지 컨버터 중에서, 대상 클래스 타입과 미디어 타입을 모두 체크하여 사용여부가 결정된다. 만약 만족하지 않으면 다음 순위 메시지 컨버터로 검증

- 주요 메세지 컨버터
	- ByteArrayHttpMessageConverter : `byte[]`데이터를 처리
		- 클래스 타입: `byte[]`, 미디어타입 `*/*`
		- 요청 예 : `@RequestBody byte[] data`
		- 응답 예 : `@ResponseBody return byte[]` 미디어 타입(response header) `application/octet-stream`
	- StringHttpMessageConverter : String으로 데이터 처리
		- 클래스 타입: `String`, 미디어타입 `*/*`
		- 요청 예 : `@RequestBody String data`
		- 응답 예 : `@ResponseBody return "ok"` 미디어 타입 `text/plain`
	- MappingJackson2HttpMessageConverter : Json으로 처리
		- 클래스 타입: 객체나 `HashMap` 미디어타입 `application/json`
		- 요청 예 : `@RequestBody MyCustomClass data`
		- 응답 예 : `@ResponseBody return MyCustomClass` 미디어 타입 `application/json`



## 메세지 컨버터 동작 방식

### 요청 데이터 읽기
- HTTP 요청이 오고, [컨트롤러](Controller.md)에서 @RequestBody, HttpEntity 파라미터를 사용하는 경우
	- 각 메세지 컨버터가 메세지를 읽을 수 있는지 `canRead()`메서드를 호출해가며 확인
		- 대상 클래스 타입 지원 여부 확인 : @RequestBody의 대상 클래스(`byte[], String, MyCustomClass`)
		- HTTP 요청의 Content-Type 미디어 타입을 지원여부 확인 : `text/plain, application/json, */*`
	- 위 조건을 만족하는 메세지 컨버터를 찾으면, 해당 메세지 컨버터로 결정되어 컨버터의 `read()`를 호출해서 객체 생성 및 반환

### 응답 데이터 생성
- 컨트롤러에서 @ResponseBody, HttpEntity로 값이 반환됨
	- 각 메세지 컨버터를 쓸 수 있는지 `canWrite()` 메서드를 호출해가며 확인
		- 대상 클래스 타입 지원 여부 확인 : return의 대상 클래스 (`byte[], String, MyCustomClass`)
		- HTTP 요청의 Accept 헤더의 미디어 타입을 지원여부 확인 (더 정확히는 [@RequestMapping의 produces 속성](Controller.md)) : `text/plain, application/json, */*`
	- 위 조건을 만족하는 메세지 컨버터를 찾으면, 해당 메세지 컨버터로 결정되어 컨버터의 `write()`를 호출해서 응답 메세지 바디에 데이터 작성


## 메세지 컨버터 동작 위치

- [스프링 MVC](스프링%20MVC.md) 구조에서 [@RequestMapping](Controller.md)을 처리하는 핸들러 어댑터는 `RequestMappingHandlerAdapter`
- 이 어댑터의 경우 유연한 파라미터를 지원하기 위해 [ArgumentResolver와 ReturnValueHandler](ArgumentResolver와%20ReturnValueHandler.md)를 호출해서 컨트롤러가 필요로 하는 다양한 파라미터의 값을 생성 (Model, @RequestParam 등)
- 이때 @RequestBody, @ResponseBody, HttpEntity등 바디의 값들을 처리하기 위한 ArgumentResolver 들이 있는데, 이들이 MessageConverter를 호출해서 필요한 값을 변환 처리 할 수 있는 것
- 자세한 내용은 [ArgumentResolver와 ReturnValueHandler](ArgumentResolver와%20ReturnValueHandler.md) 참조

### 메시지 컨버터와 ArgumentResolver / ReturnValueHandler
- `ArgumentResolver`들을 호출하여 컨트롤러가 필요로 하는 다양한 파라미터의 값을 생성하는데, 이때  @RequestBody, HttpEntity등을 처리할 수 있도록 메세지 컨버터를 호출
	- @RequestBody를 처리하기 위해 `RequestResponseBodyMethodProcessor`라는 ArgumentResolver 사용
	- HttpEntity를 처리하기 위해HttpEntityMethodProcessor 사용
	- 이처럼 스프링은 30개 이상의 ArgumentResolver를 기본으로 제공하고, [자신만의 리졸버를 정의해서 추가](ArgumentResolver와%20ReturnValueHandler.md)할 수도 있다.
- 마찬가지로`RequestMappingHandlerAdapter`는 ReturnValueHandler 를 통해 응답 값을 변환하고 처리, 이때도 @ResponseBody, HttpEntity 등을 처리할 수 있도록 메세지 컨버터 호출
	- @RequestBody를 처리하기 위해 `RequestResponseBodyMethodProcessor`라는 ArgumentResolver 사용
	- HttpEntity를 처리하기 위해HttpEntityMethodProcessor 사용
	- 스프링은 10여개의 ReturnValueHandler를 지원 (ModelAndView, @ResponseBody, HttpEntity, String 등) 하고 [추가로 정의](ArgumentResolver와%20ReturnValueHandler.md)할 수도 있다.
