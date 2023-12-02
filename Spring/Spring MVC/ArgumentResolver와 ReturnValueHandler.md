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
- 대표적인 예시는 String으로 반환해도 View이름으로 인식하게하거나, 오브젝트를 반환해도 [메세지컨버터](HttpMessageConverter.md)를 호출해서 JSON 형태로 바꾸는 등의 역할을 함

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



## ArgumentResolver를 활용해서 @Login 어노테이션 만들기

### @Login 어노테이션 만들기
```java
package hello.login.web.argumentreslover;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Target(ElementType.PARAMETER)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface Login {  
}
```
- `@Target(ElementType.PARAMETER)` : 파라미터에만 붙이는 어노테이션
- `@Retention(RetentionPolicy.RUNTIME)` [리플렉션](../../미완성%20문서/리플렉션.md) 등을 활용할 수 있도록 런타임까지 어노테이션 정보가 남아있음

### ArgumentResolver 만들기
```java
@Slf4j  
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {  

    @Override  
    public boolean supportsParameter(MethodParameter parameter) {  
        log.info("supportsParameter 실행");  
  
        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);  
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());  
  
        return hasLoginAnnotation && hasMemberType;  
    }  
  
    @Override  
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {  
  
        log.info("resolveArgument 실행");  
  
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();  
        HttpSession session = request.getSession(false);  
        if (session == null) {  
            return null;  
        }  
  
        return session.getAttribute(SessionConst.LOGIN_MEMBER);  
    }  
}
```
- `supportsParameter()` : @Login 어노테이션이 있으면서 Member 타입에 해당되면 ArgumentResolver 작동
- `resolveArgument()` : 컨트롤러 호출 직전에 호출되어서 필요한 파라미터 정보를 생성, 여기서는 로그인 회원 정보인 member 객체 반환, 이후 [스프링 MVC](스프링%20MVC.md)가 컨트롤러의 메서드를 호출하면서, 방금 가져온 member 객체를 파라미터에 전달

### WebMvcController에 설정 추가
- [서블릿 필터](../서블릿%20필터.md)나 [스프링 인터셉터](../스프링%20인터셉터.md)를 등록했던 `WebMvcController`에 방금 만든 LoginMemberArgumentResolver 등록
```java
@Configuration  
public class WebConfig implements WebMvcConfigurer {  
    @Override  
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {  
        resolvers.add(new LoginMemberArgumentResolver());  
    }
}
```


### 등록한 @Login 어노테이션과 ArgumentResolver 사용 예시

```java
@GetMapping("/")  
public String homeLoginV3ArgumentResolver(  
        @Login Member loginMember, Model model) {  
  
    if (loginMember == null) {  
        return "home";  
    }  
  
    model.addAttribute("member", loginMember);  
    return "loginHome";  
}
```
- Member 파라미터 앞에 @Login을 붙이면 적절한 member 객체 반환