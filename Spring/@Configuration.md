# @Configuration


- AppConfig를 위한 어노테이션
- 설정을 구성한다는 뜻으로, [스프링 빈](@Bean.md) 에 등록하기 위한 메서드를 정의하는 설정 클래스에 붙이는 어노테이션
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
