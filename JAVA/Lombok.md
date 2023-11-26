---
aliases:
  - 롬복
---

# Lombok

```table-of-contents
```
## Lombok이란?
- 롬복은 Java에서 반복적으로 사용되는 [getter, setter](자바%20빈%20규약.md), [toString](../미완성%20문서/toString.md) 등의 메서드 작성 코드(boilerplate code)를 줄여주는 자바 라이브러리 
- [어노테이션](../미완성%20문서/어노테이션.md)을 통해 멤버 변수마다 생기는 getter 등을 소스코드에서 보이지 않게 하고, 대신 컴파일 과정에서 생성해서 적용될 수 있게 하는 라이브러리 (코드 작성 과정에서는 보이지 않지만 `.class` 파일에서는 코드가 생성되어 있음)
- 장점
	- 어노테이션 기반의 코드 자동생성을 통해 생산성을 향상시킨다
	- 반복되는 코드를 줄여서 가독성을 높이고 유지보수를 용이하게 한다
	- getter / setter 외에도 빌더 패턴이나 [로그 생성 어노테이션](../Spring/SLF4J.md)등 다양한 방면으로 활용 가능
- 단점
	- 직관적으로 눈에 보이는 것을 선호하는 개발자에게는 비추
	- API, 내부 동작 등을 숙지해야 함
		- @Data 나 @toString을 통해 자동 생성되는 [toString](../미완성%20문서/toString.md) 메서드의 경우 [순환 참조](../미완성%20문서/순환%20참조.md) 문제로 인해 [StackOverFlowError](../미완성%20문서/StackOverFlowError.md) 가 발생 할 수 있음 -> 주의 깊은 사용 필요

## 어노테이션 종류

- @NonNull : 해당 파라미터에 null이 들어가면 [NullPointException](../미완성%20문서/NullPointException.md)를 던짐
- @Getter / @Setter : 게터 / 세터 메서드 생성
- @toString : toString 메서드 생성
- @EqualsAndHashCode : [hashCode](hashCode) 와 [equals](equals) 를 구현
- @NoArgsConstructor : 매개 변수가 없는 기본 생성자
- @RequiredArgsConstructor : final 필드만 모은 생성자
- @AllArgsConstructor : 모든 필드를 포함한 생성자
- @Data : @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor를 함께 제공
- @Value : 모든 필드를 private final로 설정하고 클래스를 final로 설정하며 setter를 생성하지 않는다
- @Builder : 메서드 체이닝을 이용하는 static 메소드인 `builder()`를 생성
- @Cleanup : 해당 영역을 벗어날때 지정된 리소스의 close() 호출, try-catch 구문 이용
- @Log, @Log4j2, [@Slf4j](../Spring/SLF4J.md) : 로그 프레임워크를 log라는 이름의 private static final 필드로 생성

이외에도 많은 어노테이션들과 어노테이션마다 추가로 설정해 줄 수 있는 속성들이 존재



---
관련 링크 : https://projectlombok.org/