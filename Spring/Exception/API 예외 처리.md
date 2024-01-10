# API 예외 처리

```table-of-contents
```

##  API 예외 처리

- [서블릿 예외 처리](서블릿%20예외%20처리.md)의 경우 기본적으로 [스프링부트](../../미완성%20문서/SpringBoot.md)가 정해진 오류 페이지를 보여주는 식으로 구현되었지만, API의 경우 오류에 따른 API 웅답 스펙을 정하고, 이에 맞게 JSON 데이터를 내려주어야 함
- 따라서 [서블릿 예외 처리](서블릿%20예외%20처리.md)에러 발생시 [WAS](../../CS/Web/Web%20Application%20Server.md)가 다시 에러 요청을 보내는 것은 유지하되, 에러 요청을 받은 컨트롤러가 JSON 형식의 응답을 보내는 식으로 구현할 수 있다.
```java
@RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)  
public ResponseEntity<Map<String, Object>> errorPage500Api(HttpServletRequest request,  
                                                           HttpServletResponse response) {  
    log.info("API errorPage 500");  
    Map<String, Object> result = new HashMap<>();  
    Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);  
    result.put("status", request.getAttribute(ERROR_STATUS_CODE));  
    result.put("message", ex.getMessage());  
  
    Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);  
    return new ResponseEntity<>(result, HttpStatus.valueOf(statusCode));  
}
```
- `produces = MediaType.APPLICATION_JSON_VALUE` : 요청의 [HTTP header](../../미완성%20문서/HTTP%20header.md)의 Accept 헤더가 `application/json`일 때 이 [컨트롤러](../Spring%20MVC/Controller.md)가 호출되도록 하는 것
- `Map<String, Object> result` : Map에 에러정보를 담아 반환할 수 있다. [Jackson](../../미완성%20문서/Jackson.md) 라이브러리는 Map 구조를 Json으로 변환해준다.
- `Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION)` : [WAS](../../CS/Web/Web%20Application%20Server.md) (서블릿 컨테이너)는 에러 발생시 해당 에러정보를 담아서 다시 요청을 보내기 때문에 이를 요청에서 확인해볼 수 있다. 자세한건 [서블릿 예외 처리](서블릿%20예외%20처리.md) 참조


## Spring Boot 기본 오류 처리

- 스프링 부트는 API로 예외 처리하는 것 역시 기본으로 제공
- [WAS](../../CS/Web/Web%20Application%20Server.md)에서 `/error` 호출시 (/error는 기본 경로, application.properties에서 server.error.path로 수정 가능) 매핑되도록 스프링 부트가 등록해 놓은 컨트롤러는 **BasicErrorController**
- 이 BasicErrorController는 두 가지 메서드를 정의해둠
	- `errorHtml()` : [서블릿 예외 처리](서블릿%20예외%20처리.md)에서 View를 제공하는 메서드, produces (요청의 Accept 헤더) text/html 인 경우 호출
	- `error()` : 그 외의 경우에 호출되는 메서드, `ResponseEntity`로 HTTP body에 JSON으로 데이터 반환
```java
@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}

@RequestMapping  
public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}
```

- 이를 통해 나가는 메세지에 어떤 오류 정보들을 담을 건지 역시 설정에서 조정 가능
- application.properties
		- `server.error.include-exception=true` (true/false)
		- `server.error.include-message=always`
		- `server.error.include-stacktrace=always` 
		- `server.error.include-binding-errors=always`
	- 기본값이 always인 설정은 3가지 옵션 가능
		- never : 사용하지 않음
		- always : 항상 사용
		- on_param : 파라미터가 있을 때 사용
			- 개발 서버에서 디버드용으로 주로 사용
- BasicErrorController를 확장하면 JSON 메세지도 변경할 수 있으나, `@ExceptionHandler`가 제공하는 기능을 이용하는게 더 나음


## HandlerExceptionResolver

- Controller 등에서 에러 발생시 원래는 DispatcherServlet까지 도달한 후, 결국 WAS까지 전달되어 500 에러
- 그런데 DispatcherServlet 도달시, 만약 HandlerExceptionResolver가 등록되어 있다면, 해당 HandlerExceptionResolver들을 가지고 예외 복구 시도
	- 단 이때에도 인터셉터의 postHandle()은 호출되지 않음
- HandlerExceptionResolver가 복구하면, 정상 흐름으로 돌리기 때문에 WAS에서 500에러를 응답하는 일은 벌어지지 않음
- 정상흐름으로 돌리는것 뿐만 아니라, 직접 예외 관련 ModelAndView를 반환하여 렌더링하거나, response body에 에러 정보를 정리해서 응답하는 것까지도 할 수 있음 (sendError()를 통해 다시 WAS에게 /error로 요청 보내게 하는 것도 가능하지만, 굳이 두번 호출되게 하는 건 불필요 할 수 있음)


### HandlerExceptionResolver 인터페이스

```java
public interface HandlerExceptionResolver {

    ModelAndView resolveException( HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex);

}
```
- handler : 핸들러(컨트롤러) 정보
- Exception : 핸들러에서 발생한 예외



### HandlerExceptionResolver 구현

```java
@Slf4j  public class MyHandlerExceptionResolver implements HandlerExceptionResolver {

	@Override
	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

		try {
			if (ex instanceof IllegalArgumentException) {
				log.info("IllegalArgumentException resolver to 400");
				response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
				return new ModelAndView();
				}
	
		} catch (IOException e) {
			log.error("resolver ex", e);
	
		}

		return null;
	}

}
```

- `ex instanceof IllegalArgumentException` : 핸들러에서 발생한 예외 (ex)가 IllegalArgumentException인 경우에 작동
- [sendError(원하는 코드, 원하는 메세지)](원하는%20코드,%20원하는%20메세지)))))))))))) : 에러를 try/catch 로 잡은 뒤, 원하는 코드와 메세지로 sendError()를 수행시킨다.
- `return new ModelAndView()` : 원하는 에러 코드를 WAS로 보낸뒤에, 빈 ModelAndView를 리턴하는 것으로 정상 흐름으로 돌린다 (**마치 에러가 없었던 것처럼**) (뷰를 렌더링 하라는 뜻이 아님)



### HandlerExceptionResolver 등록
- WebMvcConfigurer를 통해 등록
```java
@Configuration  
public class WebConfig implements WebMvcConfigurer {  
    @Override
	public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver? resolvers) {
		resolvers.add(new MyHandlerExceptionResolver());
	}
}
```

> [!Warning] extendHandlerExceptionResolvers
> configureHandlerExceptionResolvers(..)를 사용하면 스프링이 기본으로 등록하는 ExceptionResolver가 제거되므로, extendHandlerExceptionResolvers를 쓸것


### HandlerExceptionResolver 활용

- 예외가 발생했을 때, [서블릿 예외 처리](서블릿%20예외%20처리.md)에서 처럼 [WAS](../../CS/Web/Web%20Application%20Server.md)가 다시 `/error` 로 호출을 반복하는 과정은 불필요
- HandlerExceptionResolver를 활용해서 예외가 발생 했을 때, ExceptionResolver 단에서 바로 처리할 수 있도록 하는 것이 더 효율적

#### 1. 사용자 정의 Exception 생성
```java
public class UserException extends RuntimeException {

      public UserException() {
          super();
	}

      public UserException(String message) {
          super(message);
	}

      public UserException(String message, Throwable cause) {
          super(message, cause);
	}

      public UserException(Throwable cause) {
          super(cause);
	}

      protected UserException(String message, Throwable cause, boolean
  enableSuppression, boolean writableStackTrace) {
          super(message, cause, enableSuppression, writableStackTrace);
      }

}
```

#### 2. 생성한 Exception을 처리하는 HandlerExceptionResolver 생성
```java
@Slf4j
public class UserHandlerExceptionResolver implements HandlerExceptionResolver {
	private final ObjectMapper objectMapper = new ObjectMapper();

	@Override
	public ModelAndView resolveException(HttpServletRequest request,
HttpServletResponse response, Object handler, Exception ex) {
        try {
            if (ex instanceof UserException) {
                log.info("UserException resolver to 400");
                String acceptHeader = request.getHeader("accept");
                response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                if ("application/json".equals(acceptHeader)) {
                    Map<String, Object> errorResult = new HashMap<>();
                    errorResult.put("ex", ex.getClass());
                    errorResult.put("message", ex.getMessage());
                    String result = objectMapper.writeValueAsString(errorResult);
                    response.setContentType("application/json");
                    response.setCharacterEncoding("utf-8");
                    response.getWriter().write(result);
                    return new ModelAndView();
                } else {
                    //TEXT/HTML
                    return new ModelAndView("error/500");
                }

            }
        } catch (IOException e) {

            log.error("resolver ex", e);
        }

        return null;
    }

}
```
- 내가 만든 UserException 발생시 발동
- Accept 헤더 값에 따라 JSON, HTML 응답

#### 3. WebConfig에 HandlerExceptionResolver 추가
```java
@Override
public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
  resolvers.add(new MyHandlerExceptionResolver());
  resolvers.add(new UserHandlerExceptionResolver());
}
```

- 결과적으로 예외가 발생해도 ExceptionResolver가 처리하고, 그자리에서 적절한 응답을 도와주기 때문에 WAS 입장에서는 정상처리 된 것으로 보인다.
- 서블릿 컨테이너까지 올라가서 복잡한 프로세서가 실행되는 게 아니라, 그자리에서 모두 예외를 처리할 수 있다는 게 최대 장점

## 스프링이 제공하는 ExceptionResolver

- 스프링 부트가 기본으로 제공하는 ExceptionResolver
	- `HandlerExceptionResolverComposite`에 다음 순서로 등록되어 있음
		1. `ExceptionHandlerExceptionResolver`
		2. `ResponseStatusExceptionResolver`
		3. `DefaultHandlerExceptionResolver` : 가장 낮은 우선순위



### ExceptionHandlerExceptionResolver

- ExceptionHandlerExceptionResolver는 @ExceptionHandler 어노테이션을 활용해서 간단하게 문제를 해결할 수 있다.

#### 사용 방법 1
- 컨트롤러 안에 @ExceptionHandler가 달린 메서드를 선언
- 해당 컨트롤러 안에서, **@ExceptionHandler에 지정한 Exception이 터지는 경우, 해당 메서드가 실행**
```java
@Data
@AllArgsConstructor  
public class ErrorResult {
	private String code;
	private String message;
}
```
- 에러 메세지 JSON 출력하기 편한 객체 정의

```java

@Slf4j
@RestController
public class ApiExceptionV2Controller {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e) {

        log.error("[exceptionHandle] ex", e);

        return new ErrorResult("BAD", e.getMessage());
    }
    .
    .
    .

	
}
```
- @RestController이기 때문에 ErrorResult를 JSON으로 바꿔서 message body로 반환
- 에러 발생시, ExceptionHandlerExceptionResolver가 우선적으로 해당 컨트롤러에 @ExceptionHandler가 달린 메서드가 있는지 체크
- 있는 경우 Exception을 처리하고 대신 해당 메서드 실행(정상 흐름)
	- 정상 처리이므로 WAS가 다시 내부 호출하지 않음
- WAS에 도달했을 때는 이미 정상흐름 처리가 되었기 때문에, 200 OK로 응답이 나감
- 따라서 에러 코드를 바꾸고 싶다면 @ResponseStatus(에러코드) 사용

#### 사용 방법 2
```java
@ExceptionHandler
public ResponseEntity<ErrorResult> userExHandle(UserException e) {
	log.error("[exceptionHandle] ex", e);
	ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
	return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
}
```
- @ExceptionHandler 뒤의 지정 Exception의 경우, 메서드의 파라미터와 같은 경우에는 생략 가능
- ResponseEntity로 반환하면서, 엔티티에 대신 코드를 지정하는 것도 가능

#### 사용 방법 3
```java
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
@ExceptionHandler
public ErrorResult exHandle(Exception e) {
	log.error("[exceptionHandle] ex", e); 
	return new ErrorResult("EX","내부 오류");
}
```
- **지정한 Exception은 그 하위 Exception을 모두 잡음**
- 여러 @ExceptionHandler를 두고, 특정 Exception을 먼저 잡은 후에, 만약 놓치거나 공통적으로 처리할 Exception이 있다면, 이처럼 부모 Exception으로 하나 선언한 후에 **나머지를 모두 처리하도록 할 수 있음**

#### 사용 방법 4
```java
@ExceptionHandler({AException.class, BException.class})
public String ex(Exception e) {
	log.info("exception e", e);
}
```
- 여러 예외를 한번에 지정 가능

> [!info] 파라미터와 응답
> - @ExceptionHandler의 경우 스프링의 컨트롤러의 파라미터 응답처럼 다양한 파라미터와 응답을 지정 가능
> - 자세한 파라미터와 응답을 매뉴얼 참조
> - https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler-args

#### @ControllerAdvice
- @ExceptionHandler 방식의 아직 남은 문제는, 예외 처리코드와 정상 컨트롤러 코드가 하나의 컨트롤러에 섞여 있다는점
- 이를 해결하기 위해서는 [@ControllerAdvice](@ControllerAdvice.md)를 사용할 수 있다.
- 자세한 내용은 [@ControllerAdvice](@ControllerAdvice.md) 문서 참조






### ResponseStatusExceptionResolver

- ResponseStatusExceptionResolver는 예외에 따라 [HTTP status code](../../CS/Web/HTTP%20status%20code.md)를 지정해주는 역할
- 다음 두가지 경우를 처리
	- @ResponseStatus가 달려있는 예외
	- ResponseStatusException 예외

#### @ResponseStatus
- 예외에 @ResponseStatus를 달면, HTTP 상태 코드를 변경 해줌
```java
@ResponseStatus(code=HttpStatus.BAD_REQUEST,reason="잘못된 요청 오류") 
public class BadRequestException extends RuntimeException {  
}
```
- 해당 Exception이 컨트롤러 밖으로 넘어가면 `ResponseStatusExceptionResolver`가 해당 어노테이션을 인지하고 오류 코드와 메세지를 어노테이션 속성에 담은 것으로 바꿔준다.
	- 정확히는 `ResponseStatusExceptionResolver`가 `response.sendError(statusCode, resolvedReason)`을 호출한다.
	- `sendError()`를 호출했기 때문에 WAS에서 다시 오류 페이지(`/error`)를 내부 요청
- `reason` 속성의 경우 [MessageSource](../../미완성%20문서/MessageSource.md)에서 찾는 것도 가능
	- `reason = "error.bad"`로 지정하고, messages.properties에 `error.bad=잘못된 요청 오류`로 적으면 적용됨

#### ResponseStatusException
- @ResponseStatus는 외부 라이브러리등, 개발자가 직접 변경할 수 없는 예외에 적용할 수 없고(어노테이션을 붙여줄 수 없음), 어노테이션을 사용하기 때문에 조건에 따라 동적으로 변경하는 것이 어렵다.
- 이 경우에는 ResponseStatusException을 사용할 수 있다.
```java
@GetMapping("/api/response-status-ex2")
public String responseStatusEx2() {
  throw new ResponseStatusException(HttpStatus.NOT_FOUND, "error.bad", new
IllegalArgumentException());
}
```
- ResponseStatusException은 RuntimeException을 상속받고 있는 Exception
	- 따라서 원래 Exception이 발생하는 곳에서 Exception을 try/catch 등으로 잡고, 원래 발생할 Exception 대신에 ResponseStatusException을 터뜨려 주는 방식으로 쓸 수 있다.
	- ResponseStatusException가 발생하면 @ResponseStatus와 마찬가지로 ResponseStatusExceptionResolver가 똑같이 처리해준다.
	- 인자로는 [HTTP status code](../../CS/Web/HTTP%20status%20code.md)와 메세지([MessageSource](../../미완성%20문서/MessageSource.md) 적용가능)을 받고, 세번째 인자로는 error stack trace를 위해서 원래 발생한 에러를 넣어주는 것이 가능하다. (예시에서는 새로 발생할 에러 대신에 ResponseStatusException를 적용한 경우)


### DefaultHandlerExceptionResolver

- DefaultHandlerExceptionResolver는 **스프링 내부에서 발생하는 예외**를 해결
	- 대표적으로는 파라미터 바인딩 시점에 타입이 맞지 않는 경우 발생하는 `TypeMismatchException`
		- 이 경우, 원래라면 [WAS](../../CS/Web/Web%20Application%20Server.md)까지 에러가 올라가서 결과적으로 500 에러가 반환되어야 함
		- 그러나 대부분 클라이언트가 요청을 잘못 호출해서 발생하는 에러이기 때문에 400 에러가 더 정확
		- 이때 DefaultHandlerExceptionResolver가 400 에러로 변경해준다.
	- 이처럼 DefaultHandlerExceptionResolver에는 스프링이 내부적으로 오류코드를 어떻게 처리할지에 대해 수없이 정의되어 있음
- 내부 코드를 보면 결국 DefaultHandlerExceptionResolver 역시 `response.sendError()`를 사용해서 문제를 해결
	- 결과적으로 WAS에서 다시 오류 페이지 (`/error`)를 내부 요청


