# @Transactional

```table-of-contents
```

##  @Transactional

- [스프링과 트랜잭션](스프링과%20트랜잭션.md)에서 트랜잭션을 간편하게 다루기 위해 [스프링 AOP](../미완성%20문서/스프링%20AOP.md)를 도입한 어노테이션
	- `org.springframework.transaction.annotation.Transactional`
- [스프링 AOP](../미완성%20문서/스프링%20AOP.md)를 적용하기 위한 어드바이저, 포인트컷, 어드바이스 들은 [스프링부트](../미완성%20문서/SpringBoot.md)가 이미 [스프링 빈](스프링%20빈.md)으로 [스프링 컨테이너](스프링%20컨테이너.md)에 자동 등록함
	- 어드바이저 : `BeanFactoryTransactionAttributeSourceAdvisor`
	- 포인트컷 : `TransactionAttributeSourcePointcut`
	- 어드바이스 : `TransactionInterceptor`


## 적용 예시
```java

@Slf4j  
public class MemberServiceV3_3 {  
  
    private final MemberRepositoryV3 memberRepository;  
  
    public MemberServiceV3_3(MemberRepositoryV3 memberRepository) {  
        this.memberRepository = memberRepository;  
    }  
  
    @Transactional  
    public void accountTransfer(String fromId, String toId, int money) throws SQLException {  
        bizLogic(fromId, toId, money);  
    }  
  
    private void bizLogic(String fromId, String toId, int money) throws SQLException {  
        Member fromMember = memberRepository.findById(fromId);  
        Member toMember = memberRepository.findById(toId);  
  
        memberRepository.update(fromId, fromMember.getMoney() - money);  
        validation(toMember);  
        memberRepository.update(toId, toMember.getMoney() + money);  
    }  
  
  
  
    private static void validation(Member toMember) {  
        if (toMember.getMemberId().equals("ex")) {  
            throw new IllegalStateException("이체중 예외발생");  
        }  
    }  
}
```

- `@Transactional`의 경우 메서드 / 클래스 둘다 붙여도 됨
	- 클래스에 붙일 경우, 외부에서 호출 가능한 모든 `public`메서드에 AOP 적용

## 작동 방식

- 스프링 AOP는 트랜잭션 AOP가 적용된 클래스를 기반으로 [CGLIB](../미완성%20문서/CGLIB.md)를 활용해 프록시를 만들어서 스프링 컨테이너에 대신 등록
- 이후 프록시 객체가 트랜잭션을 시작한 후에, 실제 서비스에 정의된 비지니스 로직을 호출
- 비지니스 로직이 다 끝나면 다시 프록시 객체가 필요한 트랜잭션 작업을 마치고 종료


## 선언적 트랜잭션 관리 vs 프로그래밍 방식 트랜잭션 관리

- 선언적 트랜잭션 관리(Declarative Transaction Management)
	- 해당 로직에 트랜잭션을 적용하겠다고 어딘가에 선언하면 트랜잭션이 적용되는 방식
		- @Transactional 어노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것
		- 과거에는 XML 방식으로 설정
		- 간편하고 실용적이기 때문에 대부분 선언적 트랜잭션 관리 사용
- 프로그래밍 방식의 트랜잭션 관리(programmatic transaction management)
	- [트랜잭션 매니저, 트랜잭션 템플릿](스프링과%20트랜잭션.md) 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 프로그래밍 방식
	- [스프링 컨테이너](스프링%20컨테이너.md)나 [스프링 AOP](../미완성%20문서/스프링%20AOP.md) 없이 사용할 수 있기 때문에 테스트시에 가끔 사용됨

		