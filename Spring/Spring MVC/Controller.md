---
aliases:
  - "@Controller"
  - controller
  - Controller
  - 컨트롤러
  - "@RequestMapping"
tags:
  - MVC
  - Spring
---
# Controller

```table-of-contents
```

##  Controller란?

- [MVC](MVC.md) 패턴에서 요청을 받아서 파라미터를 검증하고 비지니스 로직을 실행하는 역할, 실행 결과 데이터를 조회해서 모델에 담아 뷰(렌더링 담당)에 전달하는 역할을 담당하는 개체
	- 관련 문서
		- [MVC](MVC.md)
		- [스프링 MVC](스프링%20MVC.md)


## @Controller 어노테이션

- [스프링 MVC](스프링%20MVC.md)에서 사용하는 **어노테이션 기반 컨트롤러**를 만들 때 사용하는 어노테이션
- 보통 URI를 매핑해주는 **@RequestMapping**와 함께 사용된다.

> [!Info] @ReqeustMapping의 핸들러 매핑 및 핸들러 어댑터
> 스프링 MVC가 구현한 [어댑터 패턴](어댑터%20패턴.md)에서 말하는 핸들러 매핑과 핸들러 어댑터 (가장 우선순위가 높은 핸들러 매핑과 어댑터이기도 하다)
> - `RequestMappingHandlerMapping`
> - `RequestMappingHandlerAdapter`


## 실제 사용 예시

### 예시 코드

```java

@Controller  
@RequestMapping("/springmvc/v3/members")  
public class SpringMemberControllerV3 {  
  
    private final MemberRepository memberRepository = MemberRepository.getInstance();  
  
  
    @GetMapping("/new-form")  
    public String newForm() {  
        return "new-form";  
    }  
  
    @PostMapping("/save")  
    public String save(  
            @RequestParam("username") String username,  
            @RequestParam("age") int age,  
            Model model  
    ) {  
  
  
        Member member = new Member(username, age);  
        memberRepository.save(member);  
  
        model.addAttribute("member", member);  
        return "save-result";  
  
    }
  
  
    @GetMapping  
    public String members(Model model) {  
        List<Member> members = memberRepository.findAll();  
        model.addAttribute("members", members);  
  
        return "members";  
  
    }  
  
}
```


## 코드 분석

### @Controller / @RestController
- [@Controller](.md) 어노테이션의 경우 내부에 [@Component](../../미완성%20문서/@Component.md) 어노테이션을 포함하고 있어서 [@ComponentScan](../@ComponentScan.md)의 대상이 됨
- [스프링 MVC](스프링%20MVC.md) 에서는 어노테이션 기반 컨트롤러로 인식

> [!important]
> [스프링 부트](../../미완성%20문서/SpringBoot.md) 3.0 (스프링은 6.0) 이상에서는 클래스 레벨에 `@RequestMapping`이 있어도 스프링 컨트롤러로 인식하지 않음 -> @Controller가 붙어 있어야 스프링 컨트롤러로 인식

#### @RestController
- @Controller의 경우 반환값이 String인 경우 뷰 이름으로 인식 -> 뷰를 찾아 뷰가 렌더링됨
- @RestController의 경우에는 반환값으로 뷰를 찾으려 하지 않고, 바로 [HTTP 메세지 바디](HttpServletResponse.md) 에 바로 입력
	- @Controller + @ResponseBody



### @RequestMapping("URL")
- 요청 정보를 매핑
- 해당 URL로 요청이 오면 이 메서드가 호출됨
- 클래스 레벨과 메서드 레벨에 각각 붙여줄 수 있음
	- 두 레벨에 붙은 URL의 조합으로 요청 정보를 매핑함
	- 클래스 레벨 `@RequestMapping("/springmvc/v3/members") ` + 메서드 레벨 `@PostMapping("/save")` -> `/springmvc/v3/members/save`
- URL 속성을 배열로 해서 다중 설정도 가능
	- `{"/first-uri", "/second-uri"}`

> [!important] 
> [스프링 부트](../../미완성%20문서/SpringBoot.md) 3.0 이전에는 마지막 slash (`/`)의 유무를 가리지 않았지만(정확히는 마지막 슬래시를 지워버림), 3.0 이후 부터는 마지막 슬래시가 유지되기 때문에 각각 매핑해야 한다.
> 
> - 3.0 이전 : `/hello-basic `== `/hello-basic/` 
> - 3.0 이후 : `/hello-basic `!= `/hello-basic/` (따로 매핑 해줘야 함)

- 어노테이션 기반이기 때문에, 메서드명의 경우에는 상관없이 자유롭게 지을 수 있음

#### method 속성
- @RequestMapping의 경우 `method` 속성을 통해 허용되는 [HTTP method](../../CS/Web/HTTP%20method.md) 지정 가능
	- 지정하지 않을 경우 HTTP 메서드와 상관없이 모두 허용 (GET, HEAD, POST, PUT, PATCH, DELETE)
	- 허용되지 않는 HTTP 메서드로 URI에 요청하면, [405 Method Not Allowed](../../CS/Web/HTTP%20status%20code.md) 반한
#### @GetMapping, @PostMapping
- @RequestMapping에서 까지 함께 구분해서 매핑하는 어노테이션
- URL과 HTTP method까지 모두 일치하는 요청이 매핑됨
	- 같은 URL에 HTTP method별로 메서드를 지정하는 것도 가능
- GET외에도 HTTP 메서드 별로 어노테이션이 있음
- `@RequestMapping(method = RequestMethod.GET)`과 동일

#### 경로 변수
- @RequestMapping의 URL 경로를 템플릿화 시켜서 경로 변수 사용 가능
- `@RequestMapping(/user/{userId}/article/{articleId})`
	- `{}`안에 경로 변수로 사용한 변수명 
	- 해당 변수명은 @PathVariable로 꺼낼 수 있음

#### 특정 조건 매핑
특정 조건에 따라 매핑 여부를 결정할 수 있는 속성들
##### params, headers 속성
- 특정 파라미터의 유무, 특정 헤더의 유무 조건에 따라 매핑하는 속성
```java
@GetMapping(value = "/mapping-param", params = "mode=debug")
  public String mappingParam() {
      log.info("mappingParam");
      return "ok";
  }

 @GetMapping(value = "/mapping-header", headers = "mode=debug")
  public String mappingHeader() {
      log.info("mappingHeader");
      return "ok";
  }
```

##### consumes 속성
-  HTTP 요청의 [Content-Type](../../미완성%20문서/HTTP%20header.md) 헤더를 보고 어떤 종류의 컨텐츠 타입이 오는지에 따라 추가 매핑
- 맞지 않는 경우 [415 Unsupported Media Type](../../CS/Web/HTTP%20status%20code.md) 반환

	```java
	@PostMapping(value = "/mapping-consume", consumes = "application/json")
  public String mappingConsumes() {

      log.info("mappingConsumes");

      return "ok";
  }
	```

##### produces 속성
- HTTP 요청의 [Accept](../../미완성%20문서/HTTP%20header.md) 헤더를 보고 어떤 종류의 컨텐츠 타입을 원하는지에 따라 추가 매핑
- 맞지 않는 경우 [406 Not Acceptable](../../CS/Web/HTTP%20status%20code.md) 반환

```java
@PostMapping(value = "/mapping-produce", produces = "text/html")
  public String mappingProduces() {

      log.info("mappingProduces");

      return "ok";
  }
```



### 유연한 파라미터
- 어노테이션 기반의 컨트롤러의 경우 기존의 컨트롤러 처럼 [HttpServletRequest](HttpServletRequest.md)나 [HttpServletResponse](HttpServletResponse.md)를 직접 받을 수도 있지만, request에 포함된 파라미터 혹은 response에 전달할 데이터(모델)등만 직접 파라미터로 받는 것이 가능


```java
@RequestMapping("/user/{userId}")  
public String headers(HttpServletRequest request,  
                      HttpServletResponse response,
                      Model model,
                      @RequestParam("username") String username,
                      @ModelAttribute MyData myData,
                      @PathVariable String userId, 
                      HttpMethod httpMethod,  
                      Locale locale,  
                      @RequestHeader MultiValueMap<String, String> headerMap,  
                      @RequestHeader("host") String host,  
                      @CookieValue(value = "myCookie", required = false) String cookie) {
                      ...
                      
    }
```
#### Model 파라미터
- 스프링 MVC는 메서드의 파라미터로 [Model](../../미완성%20문서/Model.md)을 제공
- 이 Model을 통해 `addAttribute()` 등을 실행하고 자동으로 response에 담겨 View로 전해짐
#### @RequestParam
- 스프링은 [HTTP 요청](HttpServletRequest.md) 파라미터를 `@RequestParam`으로 간편하게 받을 수 있다.
	- `@RequestParam("username")` = `request.getParameter("username")`
		- 파라미터 이름과 매핑하는 변수(String username) 이름이 같다면 `("username")` 생략 가능
		- 타입이 String, int, Integer 등의 단순 타입이면 `@RequestParam`까지도 생략 가능
			- 단 `@RequestParam`을 생략한 경우 스프링 MVC는 `required=false`를 적용
	- `@RequestParam Map`, `@RequestParam MultiValueMap`으로 파라미터들을 Map으로 받을 수 있다.
		- 파라미터의 값이 1개로 확실하다면 Map, 그렇지 않다면 MultiValueMap[^1] 사용
	- GET 방식의 쿼리 파라미터와 POST Form 방식의 파라미터 모두에 적용

#### @ModelAttribute
- 요청 파라미터를 받아서 필요한 객체를 만들고, 그 객체에 값을 넣어주는 과정을 스프링에서 자동화 시켜줌
- `@ModelAttribute MyData myData`
	- MyData 객체를 생성
	- 요청 파라미터의 이름으로 MyData 객체의 프로퍼티를 찾아 setter호출 (바인딩)
		- 만약 타입등이 맞지않아서 바인딩 오류가 발생하면 `BindException` 발생
- `@ModelAttribute`도 생략가능
	- `@RequestParam`을 생략했을 때와 겹치기 때문에 스프링 내부에서 구별
		- String, int, Integer와 같은 단순 타입 -> `@RequestParam`
		- 그 외 나머지 -> `@ModelAttribute` (argument resolver로 지정해둔 타입 외)

#### @PathVariable
- @RequestMapping에서 설정한 경로 변수를 파라미터로 받을 수 있다.
	- `@RequestMapping(/user/{userId}/article/{articleId})`로 경로를 설정한 경우 :
		- `@PathVariable String userId, @Pathvariable Long articleId`로 받을 수 있음
		- 경로 변수의 이름과 파라미터의 이름이 같다면 `@PathVariable` 생략 가능

#### 기타 받을 수 있는 파라미터들
- `HttpMethod` : [HTTP method](../../CS/Web/HTTP%20method.md) 조회 (`org.springframework.http.HttpMethod`)
- `Locale` : Locale 정보 조회 ([HTTP header](../../미완성%20문서/HTTP%20header.md) 의 Locale)
- `@RequestHeader MultiValueMap<String, String> headerMap` : 모든 헤더를 MultiValueMap[^1] 형식으로 조회
- `@RequestHeader("host") String host`
	- 특정 HTTP Header 조회
	- 속성
		- 필수 값 여부 : required
		- 기본 값 속성 : defaultValue
- `@CookieValue(value = "myCookie", required = false) String cookie`
	- 특정 [Cookie](../../미완성%20문서/Cookie.md) 조회
	- 속성
		- 필수 값 여부 : required
		- 기본 값 속성 : defaultValue
		
> [!Info] required와 defaultValue
> - @RequestParam, @RequestHeader, @CookieValue등에는 required와 defaultValue 속성이 있다.
> - **required** = true
> 	- 해당 파라미터(헤더, 쿠키) 의 필수 여부
> 	- 기본값은 true (필수)
> 	- 필수인데 없는 경우에는 400 에러
> 		- 파라미터 이름만 있는데 빈문자인 경우 (`?username=`) 빈문자로 통과
> 		- 기본형 (int 등의 primitive)에 null이 들어가는 경우 500 에러 (불가능) -> Integer로 받거나 defaultValue 사용
> 	
> - **defaultValue** = "기본 값" (int의 경우에는 "-1"처럼 쓸 수 있음)
> 	- 파라미터에 값이 없는 경우 기본값을 대신 적용
> 	- 기본값이 항상 있기 때문에 required 불필요
> 	- 빈문자인 경우 (`?username=`) 에도 기본 값이 적용됨

### Request Body
- API 방식에서 주로 [JSON](../../미완성%20문서/JSON.md), XML 등으로 보내는 경우, 또는 단순 TEXT로 보내는 경우, [HTTP body](../../미완성%20문서/HTTP%20body.md)에 직접 담겨져서 데이터가 넘어옴 (POST, PUT, PATCH)
- 이렇게 직접 넘어오는 경우, 파라미터 방식과는 다르게 `@RequestParam` 및 `@ModelAttribute` 등을 사용할 수 없다. (POST 방식 중 HTML Form 형태의 경우에는 파라미터로 인정)
- 따라서 이 경우에는 다른 방식으로 데이터를 받아야 함

#### InputStream / OutputStream
- 단순 텍스트의 경우 InputStream과 OutputStream을 이용해 직접 읽고 쓸 수 있다.
```java
@PostMapping("/request-body-string-v2")  
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {  

    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);  
    log.info("messageBody={}", messageBody);  
    responseWriter.write("ok");  
}
```
- [HttpServletRequest](HttpServletRequest.md)에서처럼 직접 InputStream을 쓰는 것도 가능하지만, 스프링 MVC는 파라미터로 지원
	- InputStream(Reader) : request 바디 내용을 직접 조회
	- OutputStream(Writer) : response 바디에 직접 결과 작성

#### HttpEntity
- HttpEntity 파라미터를 통해 [HTTP header](../../미완성%20문서/HTTP%20header.md), [HTTP body](../../미완성%20문서/HTTP%20body.md)의 정보를 편리하게 조회 가능
	- 바디의 정보 직접 조회
		- 요청 파라미터를 조회하는 기능과는 관계 없음
- HttpEntity는 응답에도 사용 가능
	- 메세지 바디 정보 직접 반환
	- 헤더 정보 포함
	- return 값으로 설정되면, 뷰를 조회하지 않음

```java
@PostMapping("/request-body-string-v3")  
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {  
    String messageBody = httpEntity.getBody();  
    log.info("messageBody={}", messageBody);  
    return new HttpEntity<>("ok");  
}
```

- HttpEntity를 상속받은 객체
	- RequestEntity (요청)
		- [HTTP method](../../CS/Web/HTTP%20method.md), url 정보 추가
	- ResponseEntity (응답)
		- [HTTP status code](../../CS/Web/HTTP%20status%20code.md) 설정
		- `return new ResponseEntity<String>("Hello World", responseHeader, HttpStatus.CREATED)`
			- 바디에 String 대신에 DTO를 담는 것이 일반적




#### @RequestBody
- HttpEntity 전체가 아닌 메세지 바디 부분을 좀 더 편리하게 받아보기 위한 파라미터
```java
@ResponseBody  
@PostMapping("/request-body-string-v4")  
public String requestBodyStringV4 (@RequestBody String messageBody) throws IOException {  
    log.info("messageBody={}", messageBody);  
    return "ok";  
}
```

##### JSON
- @RequestBody를 이용하면 @ModelAttribute처럼 JSON 형식의 데이터를 객체에 매핑해서 받을 수 있다.
```java
@ResponseBody  
@PostMapping("/request-body-json-v3")  
public String requestBodyJsonV3(@RequestBody HelloData helloData) {  
  
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());  
  
    return "ok";  
}
```
>[!warning] 주의
>- @RequestBody 어노테이션을 생략하면 @ModelAttribute가 생략된 것으로 인식하니 주의
>- Content-Type이 application/json이어야만 json을 처리하는 메세지 컨버터가 동작


- HttpEntity를 이용해서 받는 것도 가능
- 응답 역시 HttpEntity나 @ResponseBody를 이용해서 객체 -> JSON으로 응답 가능
```java
@ResponseBody  
@PostMapping("/request-body-json-v5")  
public HelloData requestBodyJsonV5(HttpEntity<HelloData> httpEntity) {  
    HelloData data = httpEntity.getBody();  
  
    log.info("username={}, age={}", data.getUsername(), data.getAge());  
  
    return data;
    // return new HttpEntity<>(data, HttpStatus.OK);
}
```


> [!info] HTTP 메세지 컨버터
> 
> 메세지 바디를 읽을 때 알아서 문자나 객체로 변환해 주는 것은 [HttpMessageConverter](../../미완성%20문서/HttpMessageConverter.md) 를 스프링 MVC에서 사용하기 때문
> 



### return

#### return 뷰이름;
- 어노테이션 기반의 컨트롤러는 ModelAndView(`org.springframework.web.servlet.ModelAndView`)를 직접 반환해도 되지만, String으로 View의 논리 이름만 반환해도 된다. (유연한 설계)
	- 논리 이름만 반환시, 알아서 뷰의 이름으로 판단하고 해당 뷰를 렌더링한다.
	- 만약 @ResonseBody가 붙어있는 메서드였다면 그냥 바디에 문자열이 나간다.
	- void를 반환하면서, @Controller를 사용하고, HTTP 메세지 바디를 처리하는 파라미터(HttpServletResponse, OutputSream등)가 없는 경우에는 요청 URL을 참고해서 논리 뷰 이름으로 사용 (명시성이 떨어짐)


#### return HttpEntity
- 뷰 템플릿을 사용하지 않고 메세지 바디 정보를 직접 반환 하고 싶을 때 사용
- [HttpMessageConverter](../../미완성%20문서/HttpMessageConverter.md)를 사용해서 변환되는 것
```java
@PostMapping("/request-body-json-v5")  
public HttpEntity<MemberData> requestBodyJsonV5(HttpEntity<MemberData> httpEntity) {  
    MemberData data = httpEntity.getBody();  
  
    log.info("username={}, age={}", data.getUsername(), data.getAge());  
  
    return new HttpEntity<>(data, HttpStatus.OK);
}
```


#### @ResponseBody
- 메서드 위에 붙이는 것으로 뷰 템플릿을 사용하지 않고 메세지 바디 정보를 직접 반환 가능
- 클래스 레벨에 붙이는 경우 전체 메서드에 적용
	- @RestController에 @ResponseBody가 포함되어 있음
```java
@ResponseBody  
@PostMapping("/request-body-json-v5")  
public HelloData requestBodyJsonV5(HttpEntity<HelloData> httpEntity) {  
    HelloData data = httpEntity.getBody();  
  
    log.info("username={}, age={}", data.getUsername(), data.getAge());  
  
    return data;
}
```

##### @ResponseStatus
- @ResponseBody 대신에 @ResponseStatus(HttpStatus.OK)를 이용해서 쉽게 [HTTP status code](../../CS/Web/HTTP%20status%20code.md) 설정 가능
- 다만 어노테이션이기 때문에, 응답코드를 동적으로 변경하는 건 불가능
	- 응답코드의 동적 변경이 필요한 경우에는 ResponseEntity를 사용



[^1]: 같은 키에 여러 밸류가 들어가는 경우 사용하는 것이 `MultiValueMap`, 리스트로 조회된다.  `List<String> values = MultiValueMap.get(key);`



















