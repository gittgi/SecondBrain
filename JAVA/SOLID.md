# SOLID

- Single Responsibility Principle (SRP, 단일 책임 원칙)
	- 한 클래스는 하나의 책임만 가질 것
		- 책임의 정의는 맥락과 상황에 따라 다를 수 있기에, 딱 정의하기 어렵다.
		- 따라서 변경 사항이 있을 때, 이 변경으로 인한 파급효과를 최소화 되도록 책임을 한정시키는 것이 핵심
	- 예시로는 UI 변경으로 인해 기능적인 수정이 생기지 않도록 렌더링 관련 클래스의 책임을 한정하거나, 별도의 객체를 생성하는 객체와 사용하는 객체를 분리해서 생성 객체는 생성하는 책임만, 사용 객체는 사용하는 책임만 지게하는 것

- Open/Closed principle (OCP, 개방-폐쇄 원칙)
	- 소프트웨어 요소는 확장에는 열려있되, 변경에는 닫혀 있을 것
		- 다형성을 활용하는 것으로 가능
			- 같은 인터페이스를 구현한 클래스들을 서로 바꿔끼는 것으로 기능을 **확장**
			- 동시에 해당 구현체 클래스들을 호출하는 부분에는 구현체가 아닌 인터페이스를 기준으로 코드를 작성하여, 구현체가 변경되더라도 코드 자체를 수정 혹은 **변경**하지 않아도 되도록 설계하라는 것
			```JAVA
			public class MemberService { 
				// private MemberRepository memberRepository = new MemoryMemberRepository(); // 기존 레포지토리
				private MemberRepository memberRepository = new JdbcMemberRepository(); // 새 레포지토리로 변경 }
```
			- 다만 이 경우에도 불가피한 코드 변경(주석 처리 및 새 코드 작성)이 요구됨 -> 따라서 객체를 생성하고, 연관관계를 맺어주는  


---
연관 링크 : [ ]
