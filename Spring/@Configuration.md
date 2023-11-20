# @Configuration


- AppConfig를 위한 어노테이션
- 설정을 구성한다는 뜻으로, [스프링 빈](../@Bean.md) 에 등록하기 위한 메서드를 정의하는 설정 클래스에 붙이는 어노테이션
- 이 config 클래스에서 각 빈에 필요한 의존관계를 주입하고 이를 [스프링 컨테이너](스프링%20컨테이너.md)에 등록할 수 있게 한다.
- 이 config 파일을 컨테이너 등록하기 위해서는 `ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class)` 와 같이 직접 등록할 수 있다.


## 예시 코드
- @Configuration 를 이용해서 빈 의존관계 설정
```java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean    
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean    
    public OrderService orderService() {
        return new OrderServiceImpl(
                memberRepository(),
                discountPolicy());
	}

    @Bean    
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean    
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }

}

```




---



## 바이트코드 조작 ([CGLIB](../CGLIB.md))

- @configuration 사용시 해당 클래스에서 사용되는 생성자는(위 예시에서 `memberRepository()`) 두번 이상 호출될 수 있는데, 그때마다 `new MemoryMemberRepository()` 와 같이 새로운 객체를 생성하는 것 처럼 보이나 사실은 [싱글톤](../CS/디자인%20패턴/싱글톤%20패턴.md)을 유지하기 위해 바이트 코드 조작이 들어간다.
- 정확히는 [CGLIB](../CGLIB.md) 기술을 이용해서 해당 객체를 상속받는 또다른 객체(예 : `AppConfig$$EnhancerBySpringCGLIB$$asklgns3423`)를 생성하고 이 객체를 스프링 빈으로 등록하는 것
- 이후 `memberRepository()`를 호출되게 되면 마치 프록시처럼 작용해서, memberRepository의 인스턴스가 스프링 빈으로 이미 등록되어 있는지 여부에 따라, 새로 생성 후 등록하거나, 이미 등록된 인스턴스를 반환하는 방식으로 싱글톤을 유지할 수 있게 해준다.
 - @Configuration 없이 [@Bean](../@Bean.md) 만으로도 스프링 빈으로 등록하는 것이 가능하지만, 대신 실제 객체를 그대로 등록하는 것이기 때문에, 위의 로직처럼 조건부로 생성 또는 반환하지 못해서 싱글톤을 보장하지 않는다.
 - 다시 말해 스프링 설정 정보는 항상 @Configuration 을 사용해야 한다.