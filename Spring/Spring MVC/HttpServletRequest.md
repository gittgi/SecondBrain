# HttpServletRequest

```table-of-contents
```
## HttpServletRequest란?

- [서블릿](../../CS/Web/서블릿.md)이 개발자를 대신해서 요청 메세지를 파싱 -> 그 결과를 `HttpServletRequest` 객체에 담아서 제공
	- 그 결과로 다음과 같은 **메시지 정보**들을 편리하게 조회 가능
		- start-line
			- HTTP 메서드
			- URL
			- 쿼리 스트링
			- 스키마, 프로토콜
		- [헤더](../../미완성%20문서/HTTP%20header.md)
		- [바디](../../미완성%20문서/HTTP%20body.md)
			- form 파라미터 형식 조회
			- message body 데이터 직접 조회
	- 뿐만 아니라 여러 부가기능도 함께 제공
		- **임시 저장소** : 해당 HTTP 요청의 시작부터 끝까지 유지되는 임시 저장소
			- 저장 : `request.setAttribute(name, value)`
			- 조회 : `request.getAttribute(name)`
		- **[세션](../../CS/Web/Session.md) 관리** 
			- `request.getSession(create: true)`를 통해 세션 생성


### start-line 및 헤더 조회

```java
@WebServlet(name = "requestHeaderServlet", urlPatterns = "/request-header")  
public class RequestHeaderServlet extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  

	// request에서 꺼내볼 수 있는 정보들
	// start-line
	request.getMethod(); // GET, POST
	request.getProtocol(); // HTTP/1.1
	request.getScheme(); // http
	request.getRequestURL(); // http://localhost:8080/request-header
	request.getRequestURI(); // request-header
	request.getQueryString(); // username=gittgi
	request.isSecure(); // false (HTTPS 사용여부)

	// header 
	request.getHeaderNames(); // 헤더 이름들 (Enumeration<String>)
	request.getHeader("헤더 이름"); // 헤더 이름에 해당하는 헤더 내용
    
    // 헤더 편의 조회
    request.getServerName(); // localhost (호스트)
    request.getServerPort(); // 8080 (포트)
    request.getLocales(); // Accept-Language 헤더에 설정된 언어 목록 (Enumeration<Locale>)
    request.getLocale(); // ko (Accept-Language 헤더에 설정된 가장 선호하는 언어)

	// 쿠키 조회
	if (request.getCookies() != null) {
		//request.getCookies().getCookie("쿠키 이름")으로 쿠키 클래스 꺼내기도 가능
		//request.getCookies().getValue("쿠키 이름")으로 쿠키 값만 조회도 가능
		for (Cookie cookie : request.getCookies()) {
			cookie.getName(); // 쿠키 이름 
			cookie.getValue(); // 쿠키 값
		}	
	}

	// 칸텐츠 조회
	request.getContentType(); // 요청을 전송할 때 사용한 컨텐츠 타입
	reqeust.getContentLength(); // 클라이언트가 전송한 요청저보의 길이, 알 수 없는 경우 -1
	reqeust.getCharacterEncoding(); // UTF-8등, 요청 전송시 사용한 문자 인코딩

	// 기타 정보 (HTTP 메세지 정보는 아님)
	// Remote 정보
	request.getRemoteHost() // 0:0:0:0:0:0:0:1 (클라이언트 HOST 값)
	request.getRemoteAddr() // 0:0:0:0:0:0:0:1 (클라이언트 IP 값)
	request.getRemotePort() // 54164 (클라이언트 포트 값)

	// 로컬 정보
	request.getLocalName() // localhost (서버 로컬 도메인 값)
	request.getLocalAddress() // 0:0:0:0:0:0:0:1 (서버 로컬 IP)
	request.getLocalPort() // 8080 (서버 로컬 포트)
	
    }

	

```


### 쿼리 파라미터 조회 (GET )

```java
@WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")  
public class RequestParamServlet extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        request.getParameterNames() // 파라미터 이름 목록 (Enumeration<String>)
        request.getParameter("파라미터 이름") // 파라미터 값 조회  
        System.out.println("[전체 파라미터 조회] - end");  

		// 같은 이름의 파라미터 but 복수의 값
		// username=gittgi&username=gittgi2
        String[] usernames = request.getParameterValues("username");   
  
    }  
}

```

- [!] 파라미터 이름은 하나인데 값이 복수인 경우 (`username=gittgi&username=gittgi2`) `request.getParameter()`는 맨 첫 값만 반환, 모든 값을 조회하고 싶으면 `request.getParameterValues()`를 사용

### HTML Form의 쿼리 파라미터 조회 (POST)
- `content-type: application/x-www-form-urlencoded`
- 쿼리 파라미터 형식은 동일하지만 (`username=gittgi`) [HTML](../../미완성%20문서/HTML.md) Form 형식으로 전달
- 쿼리 파라미터 형식이 위의 [GET](../../CS/Web/HTTP%20method.md) 방식과 같기 때문에, **조회 메서드 역시 그대로 사용 가능!**
- [*] `request.getParameter()`는 GET 쿼리 파라미터 형식과 HTML Form 형식 둘다 조회 가능
	

### 단순 텍스트 방식 조회 
- `content-type:text/plain`
- [HTTP message body](../../미완성%20문서/HTTP%20body.md)에 직접 담아서 요청 하는 경우, InputStream 을 사용해서 직접 읽을수 있음
```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")  
public class RequestBodyStringServlet extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
		// InputStream으로 읽기
        ServletInputStream inputStream = request.getInputStream();  
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);  
        System.out.println("messageBody = " + messageBody);  
        response.getWriter().write("ok");  
    }  
}
```
- 위에서 소개한 Form 역시 바디에 담겨 오기 때문에, 불편하긴 해도 이 방식으로 직접 읽는 것이 가능하긴 함

### API 메세지 바디 - JSON으로 받기
- `content-type: application/json`
- [HTTP API](../../미완성%20문서/HTTP%20API.md)에서 주로 사용하는 [JSON](../../미완성%20문서/JSON.md) 형식으로 데이터 전달

- [i] JSON 형식을 파싱하기 위해 객체 필요
```java
	@Getter @Setter // Lombok
	public class HelloData {  
		private String username;  
		private int age;  
	}
```

- 이후 messageBody의 JSON를 만들어둔 객체의 데이터 형식으로 맞춰서 ObjectMapper로 매핑 가능
```java
@WebServlet(name = "RequestBodyJsonServlet", urlPatterns = "/request-body-json") 

/** 
* {"username": "hello", "age": 20}
**/
public class RequestBodyJsonServlet extends HttpServlet {  
  
    private ObjectMapper objectMapper = new ObjectMapper();  
  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
	    // inputStream 적용 및 문자열 읽기
        ServletInputStream inputStream = request.getInputStream();  
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);  
        System.out.println("messageBody = " + messageBody);  
		// ObjectMapper로 문자열에서 키 값을 구분해 객체에 맞춰 파싱
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);  
        System.out.println("helloData.username = " + helloData.getUsername());  
        System.out.println("helloData.age = " + helloData.getAge());  
	  
        response.getWriter().write("ok");  
    }  
}
```

- 원래는 [Jackson](../../미완성%20문서/Jackson.md), Gson등의 JSON 변환 라이브러리를 추가해야 함
	- [SpringBoot](../../미완성%20문서/SpringBoot.md)의 [스프링 MVC](스프링%20MVC.md)를 사용하면 기본으로 Jackson 라이브러리(ObjectMapper)를 제공 



---
연관문서 : [HttpServletResponse](HttpServletResponse.md)




[디스패처 서블릿](../../CS/Web/서블릿.md)

