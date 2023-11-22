# @Autowired

- [@ComponentScan](@ComponentScan.md) 의 경우 직접 의존관계를 설정하는 코드를 작성하지 않기 때문에, 해당 컴포넌트 객체 안에서 [의존관계 주입](의존관계%20주입.md)을 설정할 수 있어야 한다.
- @Autowired를 사용하면, 컴포넌트 스캔 및 컨테이너 등록시에, [스프링 컨테이너](스프링%20컨테이너.md)에 등록되어 있는 같은 타입의 [스프링 빈](스프링%20빈.md)을 찾아서 @Autowired가  달린 생성자 혹은 필드의 해당 인스턴스에 주입된다. ( `getBean(MemberRepository.class)`와 같은 효과) ->  [의존관계 자동 주입](의존관계%20자동%20주입.md) 

- 생성자가 딱 하나인 경우, @Autowired 생략 가능 -> [@RequiredArgsConstructor](../JAVA/Lombok.md)와 연계해서 편리하게 의존관계 설정 가능

## 조회 빈이 2개 이상인 경우

- @Autowired의 경우 타입으로 빈을 조회한다.
	- 따라서 같은 타입의 구현체(혹은 하위타입)가 여러개 빈으로 등록되는 경우, 자동 주입시에 `NoUniqueBeanDefinitionException` 이 발생한다.
	- 구현체 타입(하위 타입) 으로 지정하여 빈을 조회하게 할 수는 있겠지만, 이는 [DIP](../JAVA/SOLID.md)에 위배 되고 유연성도 떨어짐, 또한 같은 타입의 이름만 다른 빈이 두개가 있을 경우엔 여전히 해결되지 않는다

- 해결 방법
	1. @Autowired 필드 명 매칭
		- @Autowired는 먼저 타입 매칭을 시도하고, 만약 같은 타입의 빈이 여러개 있는 경우 필드 이름 또는 파라미터 이름으로 빈 이름을 추가 매칭한다.
		```java
		@Autowired
		private DiscountPolicy rateDiscountPolicy
		// 일단 DiscountPolicy 타입인 클래스가 RateDiscountPolicy와 FixedDiscountPolicy 두개가 있다고 할때, 필드명인 rateDiscountPolicy까지 읽어서 RateDiscountPolicy를 주입해 줌
		```
	2. @Qualifier 사용
		- 빈으로 등록할 클래스위에 `@Qualifier("등록할 이름")`을 적어주고, 생성자나 수정자 주입시에 파라미터 앞에 `@Qualiier("등록한 이름")`을 적어주는 것으로 원하는 빈을 조회해서 가져올 수 있다
		- 추가적인 구분자를 부여하는 것이지, 빈 이름이 변경되는 것은 아니다
		- 만약 @Qualifier에 적힌 이름을 @Qualifier에서 못찾게 되면 추가로 빈의 실제 이름까지 검색해서 찾아오기는 하지만, @Qualifier는 @Qualifier를 찾는데만 쓰는걸 권장
		- 실제 빈 이름으로도 못찾으면 `NoSuchBeanDefinitionException`
		```java
		// 빈 등록시 @Qualifier("등록할 이름")
		@Component
		@Qualifier("mainDiscountPolicy")
		public class RateDiscountPolicy implements DiscountPolicy {
			...
		}

		// 의존관계 주입시에 @Qualifier("등록한 이름")
		// 생성자 주입
		@Autowired
		 public OrderServiceImpl(MemberRepository memberRepository,
	        @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
	             this.memberRepository = memberRepository;
			     this.discountPolicy = discountPolicy;
		}

		// 수정자 주입
		@Autowired
		public DiscountPolicy setDiscountPolicy(@Qualifier("mainDiscountPolicy")
			DiscountPolicy discountPolicy) {
		     this.discountPolicy = discountPolicy;
		 }


		```

	3. @Priamry 사용
		- @Primary를 등록할 빈에 붙이게 되면, @Autowired를 통해 여러 빈이 검색되는 상황에서 우선권을 가지게 된다.
		```java
		// 빈 등록시에 @Primary를 붙이면, 같은 타입이 검색되어도 얘가 우선 적용
		@Component
		@Primary
		public class RateDiscountPolicy implements DiscountPolicy {
			...
		}

		
		
		```

		- @Qualifier는 생성자 / 수정자 파라미터 앞에 항상 붙여주어야 하지만, @Primary는 빈 등록하는 곳에만 한번 정의하면 된다는 장점이 있다.
		- 우선순위의 경우에는 @Qualifier로 직접 불러오는 것이 @Primary보다 더 우선적으로 적용된다.
			- 따라서 가령 여러 데이터베이스를 복수로 운용하는 서비스라면, 메인으로 사용하는 데이터베이스 커넥션을 @Primary로 지정하고, 가끔 특수한 경우에 사용하는 서브 데이터베이스 커넥션은 @Qualifier를 활용해서 직접 지정해오는 것으로 사용하는 것을 권장
	
	4. 직접 어노테이션 만들기
		- @Qualifier("지정한 이름")의 경우 이름을 문자열로 넘겨야 하기 때문에, 컴파일 시점에서 오타등이 나더라도 타입체크가 되지 않는다.
		- 따라서 직접 어노테이션을 정의하는 것으로 컴파일 시점에 오류를 낼 수 있는 @Qualifier를 만들 수 있다.
		```java
		package hello.core.annotataion;
		import org.springframework.beans.factory.annotation.Qualifier;
		import java.lang.annotation.*;

		// 어노테이션 정의
		@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER,
		ElementType.TYPE, ElementType.ANNOTATION_TYPE})
		@Retention(RetentionPolicy.RUNTIME)
		@Documented		
		@Qualifier("mainDiscountPolicy") // @Qualifier이 내재되어 있음
		public @interface MainDiscountPolicy {
		}
		```
		
		```java
		// 파라미터에 붙여서 사용 -> 컴파일 시점에 오류확인 가능
		@Autowired
		public OrderServiceImpl(MemberRepository memberRepository,
                         @MainDiscountPolicy DiscountPolicy discountPolicy) {
		     this.memberRepository = memberRepository;
		     this.discountPolicy = discountPolicy;
		 }
		
		```
		- 자바에서는 어노테이션 상속의 개념이 없지만, 스프링은 어노테이션의 기능을 모아서 사용하는 것을 지원한다. 다른 어노테이션들도 이처럼 모아서 정의내릴 수 있지만, 무분별한 어노테이션 재정의는 유지보수가 더 어려워질 수 있다.

## 조회한 복수의 빈이 모두 필요한 경우

- 클라이언트에 선택에 따라 사용되는 빈이 달라지는 경우와 같이, 상황에 따라 조회한 빈이 모두 준비되어야 하는 경우가 있을 수 있다.
- 이 경우 Map이나 List로 모두 받을 수 있다.
	- Map의 경우에는 빈의 이름을 키, 빈 객체를 밸류로 받아온다.
	- List의 경우에는 리스트안에 빈 객체들을 받는다.
```java
	@RequiredArgsConstructor
	public class DiscountService{
		private final Map<String, DiscountPolicy> policyMap; // Map으로 받는 법
		private final List<DiscountPolicy> policyList; // List로 받는 법

		public int discountAmount(int originalPrice, String discountChoice) {
			DiscountPolicy chosenDiscount = policyMap.get(discountChoice)
			return chosenDiscount.discount(originalPrice)
		}
	}
	```
- 런타임 시점에 사용자의 선택에 따라 빈을 선택해서 사용할 수 있다는 점에서 의미 있는 방식




---
