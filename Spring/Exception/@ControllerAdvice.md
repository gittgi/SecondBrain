# @ControllerAdvice

```table-of-contents
```

##  @ControllerAdvice란?

- `@ControllerAdvice`는 대상으로 지정한 여러 [컨트롤러](../Spring%20MVC/Controller.md)에 [@ExceptionHandler](API%20예외%20처리.md), [@InitBinder](../../미완성%20문서/@InitBinder.md)와 같은 기능을 부여해주는 역할

## 사용법

### ControllerAdvice 객체 생성
- @ControllerAdvice 를 붙인 객체를 생성
- 해당 객체에 [@ExceptionHandler](API%20예외%20처리.md)를 달아 구현한 메서드들을 추가
```java
@Slf4j
@RestControllerAdvice
public class ExControllerAdvice {

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
- `@ControllerAdvice`에 대상을 지정해주지 않으면 모든 컨트롤러에 글로벌 적용
- `@RestControllerAdvice`의 경우에는 `@ControllerAdvice`와 같고 대신 `@ResponseBody`가 추가

- 이렇게 하면 이제 모든 컨트롤러에서 [@ExceptionHandler](API%20예외%20처리.md)가 공통으로 적용됨

### 대상 컨트롤러 지정 방법

- 모든 컨트롤러를 지정하는 대신, 컨트롤러나 패키지 단위로 따로 지정하고 싶은 경우 @ControllerAdvice에 속성으로 지정가능[^Link]
```java
// @RestController가 달린 모든 컨트롤러 지정
@ControllerAdvice(annotations = RestController.class) 
public class ExampleAdvice1 {} 

// 지정한 패키지 및 하위 패키지의 컨트롤러 
@ControllerAdvice("org.example.controllers") 
public class ExampleAdvice2 {} 

// 해당 인터페이스 / 추상 클래스를 구현하는 모든 컨트롤러
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class}) 
public class ExampleAdvice3 {}

```


> [!Important] 
> @ExceptionHandler + @ControllerAdvice로 예외를 깔끔하게 해결 가능


[^Link]: 자세한 스펙은 https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice 참조