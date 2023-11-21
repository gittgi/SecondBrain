# @Autowired

- [@ComponentScan](@ComponentScan.md) 의 경우 직접 의존관계를 설정하는 코드를 작성하지 않기 때문에, 해당 컴포넌트 객체 안에서 [의존관계 주입](의존관계%20주입.md)을 설정할 수 있어야 한다.
- @Autowired를 사용하면, 컴포넌트 스캔 및 컨테이너 등록시에, [스프링 컨테이너](스프링%20컨테이너.md)에 등록되어 있는 같은 타입의 [스프링 빈](../스프링%20빈.md)을 찾아서 @Autowired가  달린 생성자 혹은 필드의 해당 인스턴스에 주입된다. ( `getBean(MemberRepository.class)`와 같은 효과)





---
