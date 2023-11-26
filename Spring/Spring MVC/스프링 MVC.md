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

> [!info] [어댑터 패턴](어댑터%20패턴.md)에서의 구조와 Spring MVC 구조 비교

| 어댑터 패턴 구조                     | Spring MVC         |
| ------------------------------------ | ------------------ |
| [FrontController](프론트%20컨트롤러%5C) | DisapatcherServlet |
| URI에 맞는 핸들러 조회               | HandlerMapping     |
| 핸들러 어댑터                        | HandlerAdapter     |
| ModelView                            | ModelAndView       |
| viewResolver                         | ViewResolver       |
| MyView                               | View               |



## DispatcherServlet

> [!Important] 스프링 MVC에서의 [FrontController](프론트%20컨트롤러.md)

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