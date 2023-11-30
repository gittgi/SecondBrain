---
aliases:
  - 쿠키
---

# Cookie

```table-of-contents
```

##  Cookie란?

- TODO: 추후 추가





## 쿠키 사용

### 쿠키 생성
```java
Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
response.addCookie(idCookie);
```
- 키와 밸류로 Cookie를 만든 후에, [HttpServletResponse](../../Spring/Spring%20MVC/HttpServletResponse.md) 에 담을 수 있다.


### 쿠키 조회
```java
@GetMapping("/")
public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {
	...      
}
```
- `@CookieValue`를 통해 쿠키 조회 가능 ([HTTP header](../../미완성%20문서/HTTP%20header.md)에서 직접 꺼낼 필요 없음)

### 쿠키 만료 설정
```java
@PostMapping("/logout")
  public String logout(HttpServletResponse response) {

      expireCookie(response, "memberId");

      return "redirect:/";
  }

  private void expireCookie(HttpServletResponse response, String cookieName) {
      Cookie cookie = new Cookie(cookieName, null);
      cookie.setMaxAge(0);
      response.addCookie(cookie);

}

```
- `setMaxAge`를 통해 만료 설정 가능 (0 이면 즉시 만료)



## 쿠키의 보안 문제

- 쿠키 값은 임의로 변경 가능
	- 클라이언트에서 강제로 변경 가능
	- 브라우저에서 확인 가능
- 쿠키에 보관된 정보는 훔쳐갈 수 있다
	- 개인정보, 민감정보 등은 쿠키에 보관하기 부적절
		- 로컬에서 탈취될 수도 있고, 네트워크 전송 구간에서 탈취될 수 도 있다.
- 쿠키는 기본적으로 한번 정보를 가져가면 계속 사용 가능

### 대안
- 쿠키에 중요 값을 넣지 않는다.
	- 대신 예측 불가능한 임의의 토큰을 넣고, 서버에서 토큰과 사용자 id를 매핑해서 인식
	- 또한 서버에서 토큰을 관리
- 토큰은 해커가 임의의 값을 넣어도 맞출 수 없도록 예상 불가능한 값으로(랜덤)
- 토큰을 탈취해도 시간이 진면 사용할 수 없도록, 서버에서 해당 토큰의 만료 시간을 짧게 유지 및 탈취 의심정황 발생시 해당 토큰 강제 제거
- 즉, [Session](Session.md) 방식 도입