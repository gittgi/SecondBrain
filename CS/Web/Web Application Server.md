---
aliases:
  - WAS
---

# Web Application Server

```table-of-contents
```

## Web Application Server (WAS) 란?
- 정적 컨텐츠에 초첨을 맞춘 [웹 서버](Web%20Server.md) 와는 달리, DB 조회나 다양한 로직 처리를 요구하는 **동적인 컨텐츠**를 제공하기 위해 만들어진 어플리케이션 서버
- [HTTP](HTTP.md) 를 통해 컴퓨터나 장치에 애플리케이션을 수행해주는 미들웨어(소프트웨어 엔진)
- **웹 컨테이너(Web Container)** 또는 **[서블릿 컨테이너](서블릿.md) (Servlet Container)** 라고도 불림
	- Container는 JSP, Servlet을 실행시킬 수 있는 소프트웨어를 뜻함
	- WAS는 JSP, Servlet 구동 환경을 제공
- WAS는 웹 서버의 기능들을 구조적으로 분리하여 처리하려는 목적
	- 주요 기능
		- 프로그램 실행 환경과 DB 접속 기능 제공
		- 여러 트랜잭션의 관리
		- 비지니스 로직 수행
- [Tomcat](../../미완성%20문서/Tomcat.md)이나 jetty, Undertow 등이 대표적

## 웹 서버와 웹 어플리케이션 서버의 차이

- 웹 서버는 정적 리소스, WAS는 어플리케이션 로직에 초점
	- 그러나 두 경계가 모호해짐, 웹 서버도 기능을 담당하기도 하고, 어플리케이션 서버도 웹서버의 기능을 제공함
	- 그럼에도 두 서버를 구분해서 사용해야 하는 이유
	1. 기능 분리를 통한 서버 부하 감소
		- WAS만 가지고 모든 정적 컨텐츠를 제공하는 역할까지 부담하기에는 빠른 로직 처리 기능 수행에 부담이 될 수 있음
			- 또한 WAS에 오류 발생시, 정적인 장애 화면 노출하는 것 조차 불가능해짐 
		- 웹 서버만 사용해서 정적 컨텐츠 제공방식만 사용하기에는 다양한 요청에 따른 수많은 결과값을 모두 미리 서비스해야하는데, 자원이 부족
		- 따라서 웹 서버를 WAS 앞단에 위치시켜서 정적 리소스 요청의 대응은 웹 서버 단에서 모두 처리하고, 로직 등의 동적 처리는 WAS에 요청을 위임하는 방식으로 구현하는 것이 일반적 (클라이언트 -> 웹 서버 -> WAS -> DB)
	2. 물리적 분리를 통한 보안 강화
		- [SSL](../../미완성%20문서/SSL.md) 도입에 따른 암호화 / 복호화 처리를 웹 서버에 위임할 수 있음
	3. 여러 대의 웹 서버 및 WAS 연결 가능
		- 리소스 요청 경향에 따라서, 정적 리소스가 많이 사용되는 경우에는 웹서버를, 애플리케이션 리소스 소요가 많은 경우에는 WAS를 증설하는 방식으로 대응 가능
		 - 웹 서버는 정적 리소스만 제공하기 때문에 시스템 오류가 생길 확률이 WAS보다 적음 -> WAS나 DB에서 장애 발생시, 웹 서버단에서 클라이언트에게 오류 안내 화면을 제공할 수 있음
		 - 혹은 복수의 웹 서버 및 WAS를 연결하는 경우 오류가 발생한 WAS만 중단하고 다른 WAS로 돌리는 방식으로 무중단 운영의 장애 극복도 가능





---
