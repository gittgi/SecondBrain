---
aliases:
  - HandleMethodArgumentResolver
  - ArgumentResolver
  - ReturnValueHandler
  - HandlerMethodReturnValueHandler
---

# ArgumentResolver와 ReturnValueHandler

```table-of-contents
```

##  ArgumentResolver란?

- 정확히는 HandleMethodArgumentResolver
- [스프링 MVC](스프링%20MVC.md) 구조에서  [@RequestMapping](Controller.md)을 처리하는 핸들러 어댑터인 `RequestMappingHandlerAdapter`는 컨트롤러로 하여금 다양하고 유연한 파라미터([HttpServletRequest](HttpServletRequest.md), [Model](../../미완성%20문서/Model.md), @ModelAttribute, HttpEntity, @RequestParam 등등)를 지원해주기 위해서 여러 **ArgumentResolver**들을 호출하여 값을 생성함
- 컨트롤러가 요구하는 파라미터 목록에 맞춰 적합한 ArgumentResolver를 호출하고, 데이터들이 모두 준비가 되면 컨트롤러를 호출하면서 준비해둔 값을 넘겨줌

### 동작 방식

```java
public interface HandlerMethodArgumentResolver {

    boolean supportsParameter(MethodParameter parameter);

	@Nullable
    Object resolveArgument(MethodParameter parameter, @Nullable
    ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```

- 각 ArgumentResolver들의 `supportsParameter()` 를 호출해서, 해당 파라미터를 지원하는지 먼저 체크
- 지원한다면 `resolveArgument()`를 호출해서 실제 객체를 생성
- 이렇게 생성된 객체가 컨트롤러 호출시 같이 넘어가게 되는 것


## ReturnValueHandler란?

- 정확히는 HandlerMethodReturnValueHandler
- ArgumentHandler가 요청의 파라미터들을 처리한다면, ReturnValueHandler는 응답 값을 변환하고 처리

### 동작 방식

```java
public interface HandlerMethodReturnValueHandler {  
  
    boolean supportsReturnType(MethodParameter returnType);  
  
    void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception;  
  
}
```

- ArgumentResolver와 마찬가지로 supportsReturnType을 통해 반환 타입을 체크하고, 반환 타입이 맞다면 해당 ReturnValueHandler의 `handleReturnValue()`를 호출해서 반환값을 처리하는 방식


## 기능 확장

- ArgumentResolver, ReturnValueHandler (+ [HttpMessageConverter](HttpMessageConverter.md)) 모두 인터페이스가 제공되기 때문에, 필요에 따라서 직접 구현하고 등록하여 쓸 수 있다. (자주 쓰지는 않음)
- 등록시에는 WebMvcConfigurer 를 상속받은 객체를 만들어 [스프링 빈](../스프링%20빈.md)으로 등록하고, 그 안에 추가하면 된다.

### WebMvcConfigurer 확장 예시
```java
@Bean
public WebMvcConfigurer webMvcConfigurer() {

	return new WebMvcConfigurer() {
		@Override
		public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
			//...
		} 
		@Override
		public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
		
			//...
		
		} 
	};

}
```

