# HttpServletResponse

```table-of-contents
```
## HttpServletResponse란?
- [서블릿](../../CS/Web/서블릿.md)이 HTTP 응답 메세지를 생성 -> 개발자는 해당 응답 메세지에 들어갈 속성과 내용을 HttpServletResponse를 통해 편리하게 설정 가능
 
### 역할

- HTTP 응답 메세지 생성
	- [HTTP 응답 코드](../../CS/Web/HTTP%20status%20code.md) 지정
	- [HTTP header](../../미완성%20문서/HTTP%20header.md) 생성
	- [HTTP body](../../미완성%20문서/HTTP%20body.md) 생성
- 편의 기능
	- Content-Type 
	- [Cookie](../../미완성%20문서/Cookie.md)
	- Redirect


### 사용

```java
@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")  
public class ResponseHeaderServlet extends HttpServlet {  
  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        // 응답 코드  
        response.setStatus(HttpServletResponse.SC_OK);  
  
        // 헤더 생성 
        response.setHeader("Content-Type", "text/plain;charset=utf-8");  
        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");  
        response.setHeader("Pragma", "no-cache");  
        response.setHeader("my-header", "hello");  
  
        // Content-Type
        //response.setHeader("Content-Type", "text/plain;charset=utf-8"); 도 가능
        response.setContentType("text/plain");  
		response.setCharacterEncoding("utf-8");
		// response.setContentLength(2); //(생략시 자동 생성) 
  
        // Cookie 
        // response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600"); 도 있음
        Cookie cookie = new Cookie("myCookie", "good");
        cookie.setMaxAge(600); //600초  
	    response.addCookie(cookie);  
  
        // 리다이렉트 
        //response.setStatus(HttpServletResponse.SC_FOUND); //302  
		//response.setHeader("Location", "/basic/hello-form.html");  
        response.sendRedirect("/basic/hello-form.html");
  
        PrintWriter writer = response.getWriter();
        writer.println("ok"); // 단순 텍스트 응답
        
    }

```


## 응답 데이터 형식에 따른 방법
### 단순 텍스트 응답
- [b] 단순 텍스트 응답의 경우 위에서 소개한 코드에 마지막 줄 처럼
	- `writer.println("텍스트 내용");` 으로 응답 가능

### HTML 응답
- [b] HTTP 응답으로 HTML을 반환할 때는 content-type을 `text/html`로 설정
```java
@WebServlet(name = "ResponseHtmlServlet", urlPatterns = "/response-html")  
public class ResponseHtmlServlet extends HttpServlet {  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        //Content-Type: text/html;charset=utf-8  
        response.setContentType("text/html");  
        response.setCharacterEncoding("utf-8");  
  
        PrintWriter writer = response.getWriter(); // Html 태그를 직접 그리는 방식
        writer.println("<html>");  
        writer.println("<body>");  
        writer.println("<div>안녕?</div>");  
        writer.println("</body>");  
        writer.println("</html>");  
  
    }  
}
```


### JSON 응답
- [JSON](../../미완성%20문서/JSON.md) 으로 응답하는 경우 content-type은 `application/json`으로 설정
- [Jackson](../../미완성%20문서/Jackson.md) 이 제공하는 `objectMapper.writeValueAsString()` 을 사용하면 객체를 JSON 문자열로 쉽게 변경 가능
```java
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")  
public class ResponseJsonServlet extends HttpServlet {  
    ObjectMapper objectMapper = new ObjectMapper();  
    @Override  
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        //Content-Typ : application/json  
        response.setContentType("application/json");  
        response.setCharacterEncoding("utf-8");  
  
        HelloData helloData = new HelloData();  
        helloData.setUsername("kim");  
        helloData.setAge(20);  
  
        String result = objectMapper.writeValueAsString(helloData);  
        response.getWriter().write(result);  
  
  
    }  
}
```


- [i] `application/json`의 경우 스펙상 utf-8 형식을 사용하도록 이미 정의되어 있어서, 따로 `charset=utf-8`같은 추가 파라미터가 필요하지도 않고 지원하지 않기 때문에, `application/json` 이라고만 사용해야 함 (`application/json;charset=utf-8`의 경우에는 의미없는 파라미터를 추가한 셈)
	- `response.getWriter()`를 사용하면 인코딩 정보에 대한 추가 파라미터를 자동으로 추가해버리지만, `response.getOutputStream()`[^1]으로 변환해서 출력하면 자동으로 추가하지 않음


[서블릿](../../CS/Web/서블릿.md)


연관문서 : [HttpServletRequest](HttpServletRequest.md)

[^1]: HTTP 응답 받에 데이터를 쓰는 단계를 더 간단하게 구현한 것, HttpServletResponse에서 데이터를 쓸 수 있는 스트림을 얻어 오고, 해당 스트림에 쓰여진 데이터가 HTTP 응답의 바디로 전송됨. 일반적으로 **파일 다운로드** 등에서 생성된 파일을 스트림으로 변환 후 `response.getOutputStream()`로 얻어온 스트림에 쓰면, 해당 응답을 받은 브라우저에서 다운로드 가능