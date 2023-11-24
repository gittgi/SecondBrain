# JSP (Java Server Pages)

## JSP 란?
- [HTML](HTML.md) 코드에 JAVA 코드를 넣어서 동적 웹페이지를 생성해주는 웹 어플리케이션 도구
	- JSP 실행시 [서블릿](../CS/Web/서블릿.md)으로 변환되어 [WAS](../CS/Web/Web%20Application%20Server.md)에서 동작
	- WAS에서 데이터 처리를 거치면, 해당 데이터 + 웹페이지를 클라이언트에 응답
- [!] JSP와 서블릿은 동적 페이지를 응답하기 위한다는 점에서 동일하지만
	 - [c] 서블릿의 경우에는 자바 코드 내에 HTML 코드를 박아야 하기 때문에 불편
	- [p] JSP는 HTML 내부에 JAVA 코드가 들어가기 때문에, HTML 코드를 작성하기 간편
	

### 작동 흐름
- 클라이언트는 서버 측에 sample.jsp 파일을 요청
- 서버에 있는 jsp 컨테이너[^1]는 준비된 sample.jsp 파일을 읽고, 그 내용을 바탕으로 서블릿 파일 생성
- 생성된 서블릿 파일을 컴파일 (.java -> .class)
- 컴파일된 내용을 실행해서 HTML 파일 생성 후 jsp 컨테이너에게 전달
- jsp 컨테이너는 [HTTP](../CS/Web/HTTP.md) 를 통해 생성한 HTML 파일 반환, 응답


### 코드 예시
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>  
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>  
<html>  
<head>  
    <meta charset="UTF-8">  
    <title>Title</title>  
</head>  
<body>  
<a href="/index.html">메인</a>  
<table>  
    <thead>    <th>id</th>  
    <th>username</th>  
    <th>age</th>  
    </thead>    <tbody>    <c:forEach var="item" items="${members}">  
        <tr>  
            <td>${item.id}</td>  
            <td>${item.username}</td>  
            <td>${item.age}</td>  
        </tr>    </c:forEach>  
    </tbody>  
</table>
```
- jsp 문서는 항상 `<%@ page contentType="text/html;charset=UTF-8" language="java" %>` 로 시작
- `<% ~~ %>` 부분에 자바 코드 입력
- `<%= ~~ %>` 부분에 자바 코드 출력


## 서블릿과 JSP의 한계

- 서블릿으로 [HTML](HTML.md) 태그를 모두 그려주는 것과 비교해서는 더 깔끔해진 면이 있지만, 비지니스 로직과 뷰 영역이 한 파일에 들어있기 때문에 유지보수에 어려움이 있을 수 있다.
- 추가로 자바 코드, [레포지토리](@Repository.md) 등의 다양한 코드들이 JSP에 노출되어 있다는 점도 문제가 될 수 있다.
- 따라서 비지니스 로직은 [서블릿](../CS/Web/서블릿.md) 처럼 다른 곳에서 처리하고, JSP는 HTML 화면을 그리는 일에 집중할 수 있도록 역할을 나누는 것이 필요했고, 이것이 [MVC](MVC.md) 패턴의 시작 

[^1]: jsp 컨테이너 역시 서블릿으로 구현된 프로그램. jsp 파일을 서블릿 소스로 변환 및 컴파일까지만 담당하고, 그 이후에 서블릿의 수행은 서블릿 컨테이너가 담당. 대부분의 WAS는 서블릿 컨테이너와 jsp 컨테이너를 모두 내장







