# @ComponentScan

- [@Configuration](@Configuration.md) 의 설정정보를 통해 [스프링 빈](스프링%20빈.md)을 등록할 수 있지만, 일일이 config 파일에 의존관계를 직접 세팅하는 일을 줄이기 위해서 컴포넌트 스캔을 통해 자동으로 스프링 빈을 등록하는 기능이 제공됨
- @ComponentScan을 설정 클래스에 달면, [@Component](../@Component.md) 어노테이션이 달린([@Configuration](@Configuration.md), [@Repository](../@Repository.md) 등 @Component 를 포함하는 클래스들도) 클래스들을 자동으로 스캔해서 스프링 빈으로 등록 

- 이때 빈 등록이 자동으로 이루어지기 때문에, [의존관계 주입](의존관계%20주입.md) 을 설정해줄 수 있는 기능이 필요한데, 이는 [@Autowired](@Autowired.md) 를 사용해서 구현할 수 있다. -> [의존관계 자동 주입](의존관계%20자동%20주입.md)

## 예시 코드
```java
package hello.core;

 import org.springframework.context.annotation.ComponentScan;
 import org.springframework.context.annotation.Configuration;
 import org.springframework.context.annotation.FilterType;

 import static org.springframework.context.annotation.ComponentScan.*;

// @Configuraion 역시 @Component를 포함하므로, 컴포넌트 스캔의 대상, 따라서 복수의 설정파일의 정보도 같이 등록되어 버릴 수 있기 때문에 excludeFilters로 스캔 대상에서 제외
 @Configuration
 @ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =
 Configuration.class)) 
 public class AutoAppConfig {
 // 아무 내용이 없어도 자동으로 스캔해서 bean으로 등록
}
```


## 등록 규칙
- @ComponentScan은 @Component가 붙은 모든 클래스를 스프링 빈으로 등록
- 등록되는 스프링 빈의 이름은 기본적으로 클래스명에서 맨 앞글자만 소문자로 바꾼 것
	- 만약 직접 이름을 지정하고 싶으면 `@Component("지정할이름")` 로 지정 가능
- 생성자 (혹은 필드) 에 [@Autowired](@Autowired.md)가 붙어있는 컴포넌트라면, 해당 생성자의 인자(혹은 필드)와 같은 타입의 스프링 빈을 찾아와서 주입해 준다.  

## 탐색 위치

- 탐색을 시작하는 위치를 지정해 줄 수 있다.
```java
// 컴포넌트 스캔 시작 위치, 해당 패키지를 포함한 하위 패키지 모두 탐색
@ComponentScan(basePackages = "hello.core")

// 복수의 패키지도 가능
@ComponentScan(basePackages = {"hello.core", "hello.service"})

// 특정 클래스가 속해 있는 패키지를 시작위치로 지정 가능
@ComponentScan(basePackageClasses = MyMarker.class) //MyMarker가 있는 패키지를 시작 패키지로 하여 스캔 시작

```
- 따로 basePackages 속성을 주지 않으면 해당 @ComponentScan이 붙은 설정 클래스가 패키지의 시작위치
- 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것이 적절
	- [SpringBoot](../SpringBoot.md)의 경우 이 방식 채택 ([@SpringBootApplication](../@SpringBootApplication.md) 어노테이션이 @ComponentScan을 포함하고 있다.)


## 컴포넌트 스캔 기본 대상

- 컴포넌트 스캔은 다음의 어노테이션을 컴포넌트 스캔의 기본 대상으로 삼는다.
	- [@Component](../@Component.md) : 컴포넌트 스캔에 사용
	- [@Controller](../@Controller.md) : [스프링 MVC](../스프링%20MVC.md) 컨트롤러에서 사용 (+ 스프링 MVC 컨트롤러로 인식)
	- [@Service](../@Service.md) : 스프링 비지니스 로직을 다루는 서비스에서 사용 (+ 추가 기능은 없고, 개발자들이 핵심 비지니스 로직 계층을 인식하도록 표시)
	- [@Repository](../@Repository.md) : 스프링 데이터 접근 계층에서 사용 (+ 데이터 계층의 예외를 스프링 예외로 변환)
	- [@Configuration](@Configuration.md) : 스프링 설정 정보에서 사용 (+ 스프링 빈이 [싱글톤](../CS/디자인%20패턴/싱글톤%20패턴.md)으로 유지되도록 추가 처리)
- 위 어노테이션들은 @Component를 포함하고 있어 컴포넌트 스캔의 대상이 된다.
	- 어노테이션은 기본적으로 상속관계가 없지만 스프링은 상속과 유사하게 동작할 수 있도록 지원
- (`useDefaultFilters` 옵션을 끄면 기본 스캔 대상들이 제외됨, 기본 값은 켜져있다.)

## 필터

- `includeFilters` : 컴포넌트 스캔 대상을 추가로 지정
- `excludeFilters` : 컴포넌트 스캔에서 제외할 대상을 지정

-  컴포넌트 스캔 대상 어노테이션 생성 예제
```java
package hello.core.scan.filter;
 import java.lang.annotation.*;

 @Target(ElementType.TYPE)
 @Retention(RetentionPolicy.RUNTIME)
 @Documented
 public @interface MyIncludeComponent {
 }
```

- 컴포넌트 스캔 제외 어노테이션 생성 예제
```java
package hello.core.scan.filter;
 import java.lang.annotation.*;

 @Target(ElementType.TYPE)
 @Retention(RetentionPolicy.RUNTIME)
 @Documented
 public @interface MyExcludeComponent {
 }
```

- 대상 / 제외 어노테이션 필터 예제
```java

@Configuration
@ComponentScan(
    includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
    excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =
MyExcludeComponent.class))
static class ComponentFilterAppConfig {

} 
```
---

- FilterType 옵션
	- ANNOTATION : 기본값, 어노테이션을 인식
	- ASSIGNABLE_TYPE : 지정한 타입 + 자식타입을 인식해서 동작
	- ASPECTJ : [AspectJ](../AspectJ.md) 패턴 사용 (ex. `org.example..*Service+`)
	- REGEX : 정규 표현식 (ex. `org\.example\.Default.*`)
	- CUSTOM : TypeFilter 인터페이스를 구현해서 처리 (ex. org.example.MyTypeFilter)


## 중복 등록과 충돌

- 컴포넌트 스캔에서 같은 빈 이름을 등록하려는 경우
	- 자동 빈 등록 v 자동 빈 등록
		- 컴포넌트 스캔으로 이름이 같은 빈을 등록하게 되는 경우 `ConflictingBeanDefinitionException` 발생
	- 수동 빈 등록 v 자동 빈 등록
		- 스프링에서는 수동으로 등록하려는 빈이 자동으로 등록된 빈을 오버라이딩 함
			- `Overriding bean definition for bean 'memoryMemberRepository' with a different definition: replacing` 로그가 뜸
			- 이 경우 의도되지 않은 오버라이딩일 가능성이 높기 때문에, 문제가 될 소지가 큼
		- 따라서 [스프링부트](../SpringBoot.md)에서는 충돌나면 오류가 발생하도록 기본값 설정되어 있음
			- `Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true` 로그가 뜸
			- 스프링 부트인 `CoreApplication` 을 실행하면 오류 확인 가능

`
 