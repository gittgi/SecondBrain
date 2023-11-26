---
aliases:
  - spring MVC
  - 스프링 MVC
  - Spring MVC
tags:
  - MVC
  - Spring
---

# 스프링 MVC

```table-of-contents
```

##  스프링 MVC란?

- 스프링 MVC는 [MVC](MVC.md) 패턴부터 시작해서 [프론트 컨트롤러](프론트%20컨트롤러.md) 패턴, [어댑터 패턴](어댑터%20패턴.md)의 구조를 토대로 만들어진 프레임워크
- 따라서 어댑터 패턴에서 구현한 구조와 방식이 유사하다.

> [!info] 어댑터 패턴에서의 구조와 Spring MVC 구조 비교

| 어댑터 패턴 구조                     | Spring MVC         |
| ------------------------------------ | ------------------ |
| [FrontController](프론트%20컨트롤러%5C) | DisapatcherServlet |
| URI에 맞는 핸들러 조회               | HandlerMapping     |
| 핸들러 어댑터                        | HandlerAdapter     |
| ModelView                            | ModelAndView       |
| viewResolver                         | ViewResolver       |
| MyView                               | View               |



## DispatcherServlet

> [!Important] 스프링 MVC에서의 **Front Controller** 

- `DispatcherServlet` 은 부모 클래스에서 [HttpServlet](HttpServlet.md)을 상속 받고, [서블릿](../../CS/Web/서블릿.md)으로 동작한다.
	- DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
- [스프링 부트](../../미완성%20문서/SpringBoot.md)에서는 DispatcherServlet을 서블릿으로 자동으로 등록해준다.
	- 모든 경로 `urlPatters="/"` 에 대해 매핑됨
	- 경로 매핑의 경우 더 자세한 쪽이 우선순위를 갖게 되므로,  만약 다른 서블릿을 등록하고 해당 서블릿의 url 매핑이 좀 더 자세하다면 함께 동작 할 수 있다.
- 서블릿이 호출되면 `HttpServlet`이 제공하는 `service()`함수가 호출됨
	- 부모 중에 `FrameworkServlet`에서 해당 `service()`함수가 오버라이딩 되어 있음
	- `FrameworkServlet.service()`를 시작으로 여러 메서드가 호출(GET, POST등의 메서드 확인, `DispatcherServlet.doService` 등 )되면서 최종적으로 `DispatcherServlet.doDispatch()`가 호출됨

### 코드

- (인터셉터, 예외처리 try / catch 등의 구문은 제외한 수도 코드)
```java
protected void doDispatch(HttpServletRequest request, HttpServletRespons response) throws Exception {

    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    ModelAndView mv = null;

	// 1. 핸들러 조회  
	mappedHandler = getHandler(processedRequest); 
	if (mappedHandler == null) {
	        noHandlerFound(processedRequest, response);
			return; 
	}

	//2.핸들러 어댑터 조회-핸들러를 처리할 수 있는 어댑터  
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

	// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환 
	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
	
	processDispatchResult(processedRequest, response, mappedHandler, mv,
	  dispatchException);
	
}


private void processDispatchResult(HttpServletRequest request,
HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView
mv, Exception exception) throws Exception {
	
	// 뷰 렌더링 호출  
	render(mv, request, response);

}


protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
	View view;
	String viewName = mv.getViewName(); 
	
	//6. 뷰 리졸버를 통해서 뷰 찾기,7.View 반환
	view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
	// 8. 뷰 렌더링
	view.render(mv.getModelInternal(), request, response);
}


```

### DispatcherServlet에 등록할 인터페이스
- DispatcherServlet의 경우, 코드 변경 없이 기능을 확장할 수 있도록 인터페이스를 제공하기 때문에, 해당 인터페이스들을 구현해서 DispatcherServlet에 등록하면 나만의 커스텀 프론트 컨트롤러를 만들 수도 있다.
- 교체 가능한 주요 인터페이스 목록
	- 핸들러 매핑 : `org.springframework.web.servlet.HandlerMapping`
	- 핸들러 어댑터 : `org.springframework.web.servlet.HandlerAdapter`
	- 뷰 리졸버 : `org.springframework.web.servlet.ViewResolver`
	- 뷰 : `org.springframework.web.servlet.View`
	- (그러나 스프링 MVC는 오랜시간에 걸쳐서 발전해왔고, 필요한 기능에 맞추어 대부분 다 잘 구현되어 준비되어 있다)


## 핸들러 매핑과 핸들러 어댑터

- 핸들러(컨트롤러)의 경우, 과거에는 `org.springframework.web.servlet.mvc.Controller` 인터페이스를 구현하는 것으로 만들었고
- 최근에는 [@RequestMapping](@RequestMapping.md)이나 [@Controller](../../미완성%20문서/Controller.md) 를 이용해서 만든다.
- 어느쪽이든 생성한 핸들러를 스프링 빈으로 등록하고 나면, 이제 스프링 MVC에서 Controller로 사용할 수 있다.
- 이렇게 빈으로 등록한 컨트롤러를 사용하기 위해서는 2가지가 필요


### 1. HandlerMapping(핸들러 매핑)
- 먼저 핸들러 매핑에서 방금 등록한 컨트롤러를 찾을 수 있어야 한다.
	- 핸들러 매핑에서 적합한 핸들러(컨트롤러)를 찾는 방식은 여러가지 있는데, 스프링에서 이미 필요한 핸들러 매핑들을 대부분 구현해 두었고,  [스프링부트](../../미완성%20문서/SpringBoot.md)에서 자주쓰이는 일부를 자동으로 등록해 둠
	- 대표적인 핸들러 매핑의 예 (숫자는 우선순위)
		- `0 = RequestMappingHandlerMapping` : 어노테이션 기반의 컨트롤러인 [@RequestMapping](@RequestMapping.md)에서 사용
		- `1 = BeanNameUrlHandlerMapping` : 요청받은 URL과 같은 이름으로 핸들러 찾기
			-  (이 경우에는 컨트롤러의 빈 이름을 URL과 같은 모양으로 지어야 함)
	- 우선순위에 따라 @RequestMapping을 다 검색해보고, 없을 경우 다음 순위인 빈 이름으로 검색해서 핸들러를 조회
		
### 2. HandlerAdapter(핸들러 어댑터)
- HandlerAdapter에서 순서대로 각각의 어댑터의 `supports()`메서드를 실행
	- HandlerMapping과 마찬가지로, 필요한 대부분의 어댑터는 스프링이 구현해 두었고, 스프링 부트가 자동으로 등록해두었다.
	- 대표적인 어댑터의 예 (숫자는 우선순위)
		- `0 = RequestMappingHandlerAdapter` : 어노테이션 기반의 컨트롤러인 [@RequestMapping](@RequestMapping.md)에서 사용
		- `1 = HttpRequestHandlerAdapter` : HttpRequestHandler 처리
		- `2 = SimpleControllerHandlerAdapter` : 어노테이션을 쓰지 않는 과거 버전의 `Controller` 인터페이스 처리
	- 위에서 찾은 핸들러(컨트롤러)와 호환되는 경우 해당 어댑터 실행 (`handle()` 호출)
		

## 뷰 리졸버

- 뷰 리졸버 역시 스프링이 구현 해 둔 리졸버들을 스프링 부트가 자동으로 등록해둠
- 자동 등록되는 뷰 리졸버 중 대표적인 일부
	- `1 = BeanNameViewResolver` : 빈 이름으로 뷰를 찾아서 반환 (예 : 엑셀 파일 생성용 뷰를 직접 구현한 다음 빈으로 등록해서 사용할 때)
	- `2 = InternalResourceViewResolver` : [JSP](../../CS/Web/JSP.md)를 처리할 수 있는 뷰 반환
	-  [Thymleaf](../../미완성%20문서/Thymleaf.md) 뷰 템플릿의 경우 `ThymeleafViewResolver`를 따로 등록해줘야 한다. 스프링 부트를 사용하면, 선택해서 자동으로 등록할 수 있다.

- 핸들러 어댑터의 `handle()` 메서드의 호출 결과로 논리 뷰 이름 (실제 물리적인 이름에서 경로나 확장자 등의 중복부분을 제거한 값) 을 반환
- 이 이름을 가지고 등록되어 있는 viewResolver를 순서대로 호출
	- 해당 논리 뷰 이름으로 등록된 스프링 빈이 없다면 `BeanNameViewResolver`는 통과하고, 그 다음인 `InternalResourceViewResolver`가 호출됨
- 호출한 뷰 리졸버를 통해 뷰를 반환
	- `InternalResourceViewResolver`의 경우 `InternalResourceView`를 반환 -> JSP처럼 `forward()`를 호출해서 처리할 수 있는 경우에 사용
		-  `InternalResourceViewResolver`의 경우, JSTL 라이브러리가 있으면 `InternalResourceView`를 상속받은 `JstlView`를 반환
- 뷰를 통해 렌더링
	- view.render()를 호출해서 렌더링한다.
	- JSP의 경우에는 `InternalResourceView`의  `forward()`로 JSP 실행
	- 다른 뷰 템플릿들의 경우에는 `forward()` 과정 없이 바로 실제 뷰 렌더링



## 최근 방식의 스프링 MVC (@RequestMapping과 @Controller)

- 과거 스프링 프레임워크의 경우 MVC 부분이 약하다고 평가 받으며 MVC 웹 기술은 스트럿츠 등의 다른 프레임워크를 사용
- 그러나 @RequestMapping과 같은 어노테이션 기반의 컨트롤러의 등장으로 지금은 MVC 역시 스프링 MVC 사용

> 
> 자세한 내용은 **[@RequestMapping](@RequestMapping.md)** 에서 계속
> 



