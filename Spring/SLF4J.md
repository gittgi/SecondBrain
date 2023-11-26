# SLF4J

```table-of-contents
```

##  SLF4J(Simple Logging Facade for Java)란? 

-  Logback, Log4J, Log4J2 등 다수의 로깅 라이브러리에 대한 인터페이스를 제공하는 **SLF4J** 라이브러리
- SLF4J는 인터페이스이고, 그 구현체로 Logback 등의 로그 라이브러리를 선택하는 것 (스프링 부트는 기본적으로 Logback)
- SLF4J API를 사용하여 로깅 코드 작성
- 배포시에 바인딩 된 로깅 프레임워크로 실제 로깅 수행
	- 스프링 부트 프로젝트 생성시 기본적으로 `spring-boot-starter-logging`가 설정되어 있는데, 이 라이브러리가 SLF4J와 Logback을 채용

## 로그 선언
- `private Logger log = logFactory.getLogger(getClass())`
- `private static final Logger log = LoggerFactory.getLogger(Xxx.class)`
- `@Slf4j` : [Lombok](../JAVA/Lombok.md)으로 사용 가능, 사용할 클래스 또는 메서드 위에 어노테이션을 달아서 사용


## 로그 호출
- `log.info("hello")`
- `log.info("파라미터 바인딩 됩니다 : {}", data)`
	- `log.info("파라미터 문자열 붙이기?" + data)`를 쓰지 않는 이유는, 해당 라인에서 실제 더하기 연산이 실행됨 -> 로그 레벨로 인해 찍지 않아도 되는 로그 라인에서도 연산이 수행되기 때문에 손해, 반면에 파라미터 바인딩 방식은 아무일도 일어나지 않는다.


## 로그 레벨
`application.properties`나 `application.yml` 등의 설정 파일에서 로그 출력 레벨을 설정할 수 있음
```properties
#전체 로그 레벨 설정(기본값은 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 패키지에서의 로그 레벨 설정
logging.level.hello.springmvc=trace
#trace 이하 로그는 다 보겠다는 뜻
```
- 로그 레벨의 순위는 다음과 같다.
	- `log.trace()`
	- `log.debug()` : 개발 서버 추천
	- `log.info()` : 운영 서버 추천
	- `log.warn()`
	- `log.error()`


## 로그 사용시 장점
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께볼 수 있다.
- 출력 모양을 보기 편하게 조절 할 수 있다.
- 개발 환경, 운영 환경 등에 따라서 로그 레벨을 조절할 수 있다.
- 콘솔에만 출력하는 시스템 아웃 방식에 비해, 파일이나 네트워크 등 별도의 위치에 로그 이력을 남기고 저장할 수 있다.
- 성능 측면에서도 시스템 아웃보다 좋다. (내부 버퍼링, 멀티 쓰레드 등)