---
aliases:
  - HttpSession
  - 세션
---

# Session

```table-of-contents
```

##  Session이란?

- TODO: 추후 추가



## 세션 사용 이유
- [Cookie](Cookie.md) 방식의 한계를 보완하고자 세션 도입
	- 쿠키 값을 변조 가능하다는 단점을 보완하고자, 예상 불가능한 복잡하고 랜덤한 세션 ID 값을 사용
	- 쿠키에 보관하는 정보는 클라이언트에서 탈취당할 수 있기 때문에 의미 없는 세션 ID만 클라이언트에 보관하고, 해당 세션 ID를 키 삼아서 서버에서 세션을 관리
	- 쿠키를 탈취한 후 사용할 가능성의 경우, 세션의 만료시간을 짧게 설정하여 사용 가능성을 줄이고, 의심스러운 정황이 있다면 서버에서 세션을 강제로 제거하는 방법을 사용할 수도 있다.


## HttpSession

- [서블릿](서블릿.md)은 HttpSession 기능을 제공
- HttpSession을 생성하면 [쿠키](Cookie.md)를 하나 생성
	- 쿠키 이름 : `JSESSIONID`
	- 값 : 임의의 랜덤 값

### 세션 생성 및 조회
```java
@PostMapping("/login")
public String loginV3(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request) {  
        
        // 로그인 성공 이후...
          
        // 세션이 있으면 있는 세션, 없으면 신규 세션을 생성 후 반환  
        // 이때 request에 저장되어 있는 쿠키(jsessionId)의 난수를 읽어서 저장되어 있는 세션정보가 있는지 확인한다는 뜻  
        HttpSession session = request.getSession();  
        // 세션에 로그인 회원 정보 보관  
        // SessionConst.LOGIN_MEMBER의 경우 반복되는 문자열(키 값)을 상수로 정의해둔 것
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);  
        
        return "redirect:/";  
    }
```

- `public HttpSession getSession(boolean create)`을 통해 세션 조회 및 생성 가능
- create 파라미터의 값에 따라 다른 작용 (아무것도 안넣은 기본값은 true)
	- `request.getSession(true)` : 세션이 있으면 기존 세션 반환, 없으면 새로운 세션 생성해서 반환 
	- `request.getSession(false)` : 세션이 있으면 기존 세션 반환, 없으면 새로운 세션을 생성하는 대신 null 반환
> [!important] request.getSession(false)의 사용
> - 로그인 하지 않을 사용자의 게시판 조회 요청등에도 만약 request.getSession()이 사용된다면, **무의미한 세션이 새로 만들어질 수 있다**. 따라서 로그인이 아닌, 세션을 찾아서 사용하는 시점의 경우에는 **create 파라미터의 값을 false**로 넣어주어야 한다.
- `session.setAttribute`로 데이터 보관, 하나의 세션에 여러값을 저장하는 것도 가능


### @SessionAttribute
- @SessionAttribute를 사용해서 더 편리하게 세션을 사용할 수 있다.
```java
@GetMapping("/")
public String homeLoginV3Spring(@SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model) {

	//세션에 회원 데이터가 없으면 home 
	if (loginMember == null) {
		return "home";
	}

	//세션이 유지되면 로그인으로 이동 
	model.addAttribute("member", loginMember); 
	return "loginHome";

}
```
- @SessionAttribute를 통해 파라미터로 세션에 담긴 정보를 바로 받아볼 수 있다.
- 이 기능의 경우 **세션을 생성하지는 않기 때문에**, 이미 로그인된 사용자를 찾는 용도로 사용하기에 적합하다.

### 세션 삭제
```java
@PostMapping("/logout")  
public String logoutV3(HttpServletRequest request) {  
    HttpSession session = request.getSession(false);  
    if (session != null) {  
        session.invalidate();  
    }  
    return "redirect:/";  
}
```
- `session.invalidate()` : 세션을 제거


### TrackingModes
- 로그인을 처음 시도하면, URL이 `jsessionid`를 포함하는 경우를 발견할 수 있다.
- `http://localhost:8080/;jsessionid=~~~~~`
- 이는 웹브라우저가 쿠키를 지원하지 않을 때, 쿠키 대신 URL로 세션을 유지하는 방법
	- 이 방법을 쓰려면, URL에 계속해서 값을 포함시켜 전달해야한다.
	- 타임리프 등의 템플릿 엔진은, 링크를 걸면 `jsessionid`를 URL에 자동으로 포함시켜준다.
	- 첫 시도에 항상 포함되는 이유는, 최초 시점에는 해당 브라우저가 쿠키를 지원하는지의 여부를 판단하지 못하기 때문에 일단 포함시켜보는 것
- `application.properties`에 `server.servlet.session.tracking-modes=cookie` 옵션을 주면 이제는 URL에 `jsessionid`가 포함되지 않는다.


### 세션 정보 확인

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Date;

@Slf4j
@RestController  public class SessionInfoController {
	
	@GetMapping("/session-info")
	public String sessionInfo(HttpServletRequest request) {

		HttpSession session = request.getSession(false);
		if (session == null) {
			return "세션이 없습니다."; 
		}

		//세션 데이터 출력 
		session.getAttributeNames().asIterator()
			.forEachRemaining(name -> log.info("session name={}, value={}", name, session.getAttribute(name)));

		log.info("sessionId={}", session.getId());
		log.info("maxInactiveInterval={}", session.getMaxInactiveInterval());
		log.info("creationTime={}", new Date(session.getCreationTime()));
		log.info("lastAccessedTime={}", new

		Date(session.getLastAccessedTime()));
		log.info("isNew={}", session.isNew());

		return "세션 출력"; 
	
	}

}
```
- sessionId : `jsessionid`의 값
- maxInactiveInterval : 세션의 유효시간 (초)
- creationTime : 세션 생성일시
- lastAccessedTime : 세션과 연결된 사용자가 최근에 서버에 접속한 시간, 클라이언트에서 서버로 sessionId를 요청한 경우 갱신
- isNew : 새로 생성된 세션인지, 아니면 이미 만들어져서 조회한 세션인지 여부


### 세션의 타임아웃
- 세션은 로그아웃을 직접하는 경우 `session.invalidate()`를 통해 삭제해주면 된다.
- 그러나 대부분의 사용자는 로그아웃 하지 않고 브라우저를 끈다.
	- 브라우저의 입장에서는 세션 쿠키의 경우 삭제 처리가 되겠지만
	- 서버의 입장에서는 브라우저가 종료된 건지를 알 수 없기 때문에, 세션을 삭제할 타이밍 역시 알 수 없다.
- 세션을 무한정 보관하는 경우의 문제점
	- 세션과 관련된 쿠키 `jsessionid`를 탈취당한 경우, 오랜 시간이 지나도 해당 쿠키를 악용할  수 있다.
	- 세션은 기본적으로 메모리에 생성되기 때문에, 불필요한 세션을 지워서 메모리를 확보해야 한다.

- 따라서, 세션의 종료시점을 설정해야 하는데, 일괄적으로 30분 처럼 설정하는 경우, 30분 마다 다시 로그인을 해야하는 번거로움이 생긴다.
	- 따라서 결론은, 클라이언트의 마지막 요청으로부터 30분 처럼, 마지막 요청을 기준으로 세션 타임아웃을 갱신하는 방식을 사용하는 것 -> HttpSession의 경우에는 이 방법을 사용


#### 세션의 타임아웃 발생
- 세션의 타임아웃 시간은 해당 세션의 `JSESSIONID`를 전달하는 [Http 요청](../../Spring/Spring%20MVC/HttpServletRequest.md)이 있으면 현재 시간으로 다시 초기화 되고, 이후 타임아웃으로 설정한 시간동안 추가로 세션이 유지된다.
- `session.getLastAccessedTime()`을 통해 최근 세션 접근 시간을 확인할 수 있고, LastAccessedTime 이후로 timeout 시간이 지나면, [Web Application Server](Web%20Application%20Server.md)가 내부에서 해당 세션을 제거

#### 설정 방법
- 스프링 부트로 글로벌 설정
	- application.properties 에 `server.servlet.session.timeout=60`를 넣는다.
	- 60은 60초를 뜻하고, 기본값은 1800(30분)이다.
	- 글로벌 설정의 경우에는 분 단위로 해야 한다. (60, 120, 180, ...)
- 특정 세션 단위 설정
	- `session.setMaxInactiveInterval(1800)` 를 통해 세션 별로 직접 설정도 가능
	- 1800은 1800초 = 30분

## 세션 사용시 주의사항

- 세션 저장 용량을 효율적으로 사용하기 위해서, 세션에는 최소한의 데이터만을 저장할 것
- 세션에 사용되는 메모리 용량을 적절히 관리하기 위해서, 세션 타임아웃의 시간을 적절히 고민할 것
	- 기준을 기본값인 30분에 두고 고민할 것