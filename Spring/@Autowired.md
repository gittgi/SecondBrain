# @Autowired

- [@ComponentScan](@ComponentScan.md) 의 경우 직접 의존관계를 설정하는 코드를 작성하지 않기 때문에, 해당 컴포넌트 객체 안에서 [의존관계 주입](의존관계%20주입.md)을 설정할 수 있어야 한다.
- @Autowired를 사용하면, 컴포넌트 스캔 및 컨테이너 등록시에, [스프링 컨테이너](스프링%20컨테이너.md)에 등록되어 있는 같은 타입의 [스프링 빈](../스프링%20빈.md)을 찾아서 @Autowired가  달린 생성자 혹은 필드의 해당 인스턴스에 주입된다. ( `getBean(MemberRepository.class)`와 같은 효과) ->  [의존관계 자동 주입](의존관계%20자동%20주입.md) 

- 생성자가 딱 하나인 경우, @Autowired 생략 가능 -> [@RequiredArgsConstructor](../JAVA/Lombok.md)와 연계해서 편리하게 의존관계 설정 가능

## 조회 빈이 2개 이상인 경우

- @Autowired의 경우 타입으로 빈을 조회한다.
	- 따라서 같은 타입의 구현체(혹은 하위타입)가 여러개 빈으로 등록되는 경우, 자동 주입시에 `NoUniqueBeanDefinitionException` 이 발생한다.
	- 구현체 타입(하위 타입) 으로 지정하여 빈을 조회하게 할 수는 있겠지만, 이는 [[SOLID|DIP]]에 위배 되고 유연성도 떨어짐, 또한 같은 타입의 이름만 다른 빈이 두개가 있을 경우엔 여전히 해결되지 않는다

- 해결 방법
	1. @Autowired 필드 명 매칭
		- @Autowired는 먼저 타입 매칭을 시도하고, 만약 같은 타입의 빈이 여러개 있는 경우 필드 이름 또는 파라미터 이름으로 빈 이름을 추가 매칭한다.
		```java
		@Autowired
		private DiscountPolicy rateDiscountPolicy
		// 일단 DiscountPolicy 타입인 클래스가 RateDiscountPolicy와 FixedDiscountPolicy 두개가 있다고 할때, 필드명인 rateDiscountPolicy까지 읽어서 RateDiscountPolicy를 주입해 줌
		```
	2. @Qualifier 사용
		- 
---
