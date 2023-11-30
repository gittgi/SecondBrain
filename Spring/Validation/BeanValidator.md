# BeanValidator

```table-of-contents
```

##  BeanValidator란?

- 필드에 대한 검증 로직의 경우, 보통 빈 값인지 혹은 크기 제한을 넘는지 등의 일반적인 로직인 경우가 대부분
- 따라서 매번 [Validator](Validator.md)를 직접 구현하고 등록하는 것은 번거로울 수 있기 때문에, 모든 프로젝트에 적용할 수 있게끔 공통화하고 표준화 한 것이 Bean Validation
- BeanValidator를 이용하면 어노테이션으로 검증 로직 적용 가능

- Bean Validation은 Bean Validation 2.0(JSR-380)의 기술 표준
- 여러 검증 어노테이션들과 인터페이스들의 모음
- 주로 사용하는 구현체는 Hibernate Validator (ORM과 관련 없음)


## 등록
```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

- Jakarta Bean Validation
	- 버전 문제로 javax 가 등록이 안되는 경우 :
	- `jakarta.validation-api`이 인터페이스
	- `hibernate-validator`가 하이버네이트 구현체


## 사용 예시

```java
package hello.itemservice.domain.item;  
  
import lombok.Data;  
import org.hibernate.validator.constraints.Range;  
 
  
import javax.validation.constraints.Max;  
import javax.validation.constraints.NotBlank;  
import javax.validation.constraints.NotNull;  
  
@Data  
public class Item {  
  
    @NotNull(groups = UpdateCheck.class)  
    private Long id;  
  
    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})  
    private String itemName;  
  
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})  
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})  
    private Integer price;  
  
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})  
    @Max(value = 9999, groups = {SaveCheck.class})  
    private Integer quantity;  
  
    public Item() {  
    }  
  
    public Item(String itemName, Integer price, Integer quantity) {  
        this.itemName = itemName;  
        this.price = price;  
        this.quantity = quantity;  
    }  
}
```

- `@NotBlank` : 빈값 + 공백만 있는 경우를 허용하지 않음
- `@NotNull` : null 값을 허용하지 않음
- `@Range(min = 1000, max = 1000000)` : 범위 안의 값
- `@Max(9999)` : 최대 허용 값

>[!info] 자바 표준과 hibernate
> - `javax.validation.constraints.NotNull`의 경우 자바 표준 인터페이스
> - `org.hibernate.validator.constraints.Range`의 경우 하이버네이트 구현체가 제공하는 검증 기능


## 검증 절차
### 빈 검증기를 직접 사용하여 검증
- 먼저 factory를 통해 validator 가져오기
```java
  ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
  Validator validator = factory.getValidator();
```


- 검증 대상을 직접 검증기에 넣어 결과를 받아오기
- Set에  `ConstraintViolation` 라는검증오류가 담김 -> 비어있다면 오류가 없는 것
```java
Set<ConstraintViolation<Item>> violations = validator.validate(item);
```

- 이후 `ConstraintViolation`에 검증 오류가 발생한 객체, 필드, 메세지 등의 다양한 정보 확인 가능

> [!info] 스프링과 통합
> - 스프링과 통합시에는 직접 검증 코드를 불러와서 실행하는 일은 없음


### 스프링 MVC에서 검증

- [스프링부트](../../미완성%20문서/SpringBoot.md)가 `spring-boot-starter-validation`라이브러리를 보고 자동으로 Bean Validator를 스프링에 통합
- 스프링 부트는 자동으로 `LocalValidatorFactoryBean`을 글로벌 Validator로 등록
	- 이 Validator는 어노테이션 (@NotNull 등)을 보고 검증을 수행
	- 글로벌로 등록되었기 때문에, [@Valid, @Validated](Validator.md) 만 적용하면 끝
- 검증 오류 발생 시, FieldError, ObjectError를 생성해서 [BindingResult](BindingResult.md)에 담는다.
> [!Important] 직접 글로벌 등록 주의
> - [Validator](Validator.md)에서 처럼 글로벌로 직접 Validator를 등록하는 경우, 스프링 부트는 Bean Validator를 글로벌 Validator로 등록하지 않고, 어노테이션 기반의 빈 검증기도 동작하지 않는다.



## 검증 순서

1. [@ModelAttribute](../Spring%20MVC/Controller.md)에서 각각의 필드에 타입 변환 시도
	- 실패시 [typeMismatch](BindingResult.md) 로 FieldError 추가
2. Validator 적용

> [!info] 바인딩에 성공한 필드만 Bean Validation 적용
> - 일단 모델 객체에 바인딩 받는 값이 정상으로 들어와야 검증 
> - 타입변환까지 성공해서 바인딩이 성공한 필드여야 검증이 의미가 있다.



## 에러 코드와 메세지

- Bean Validator도 어노테이션 이름을 기반 오류코드로 해서 [MessageCodesResolver](MessageCodesResolver.md)를 통해 메세지 코드들이 생성됨
```text
//@NotBlank 예시
NotBlank.item.itemName 
NotBlank.itemName 
NotBlank.java.lang.String 
NotBlank

//@Range 예시
Range.item.price 
Range.price 
Range.java.lang.Integer 
Range

```

- 따라서 해당 코드를 기반으로 errors.properties에 추가해주면 메세지를 바꿀 수 있다.
```properties
#Bean Validation 추가 
NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용 
Max={0}, 최대 {1}
```
- `{0}`은 해당 필드명의 자리이고, `{1}`, `{2}`는 어노테이션마다 다르게 정의되어 있음

### BeanValidation 메세지 찾는 순서
1. 생성된 메세지 코드 순서대로 [MessageSource](../../미완성%20문서/MessageSource.md) (error.properties)에서 메세지 찾기
2. 어노테이션의 message 속성 사용 `@NotBlank(message = "{0} 필드에 공백은 안됩니다."`
3. 라이브러리가 제공하는 기본 값 사용 


## 오브젝트 오류

### @ScriptAssert()

- 특정 필드 오류가 아닌, 오브젝트 관련 오류의 경우, `@ScriptAssert()`를 사용할 수 있다.

```java
@Data
@ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >=
10000")
public class Item {
	...
}
```

- 이 결과 메세지 코드가 생성된다.
	- `ScriptAssert.item`
	- `ScriptAssert`

> [!info] 단점
> - 실제 사용시에는 제약도 많고 복잡하며, 검증 기능이 해당 객체 범위를 벗어나는 경우에는 @ScriptAssert()로는 대응이 어렵다.
> - 따라서 오브젝트 오류 부분의 경우에는 직접 자바 코드로 작성하여 [BindingResult](BindingResult.md)에 reject()를 통해 직접 넣어주는 것을 권장


## 한계

### 요구사항 충돌
- 상황에 따라 검증 요구사항이 달라질 수 있다.
	- 최초 등록시에는 id 값이 설정되지 않아 null 일 수 있으나, 수정시에는 id값이 꼭 필요하다.
	- 가령 같은 Item의 필드 Validation이라 할지라도 요구사항 스펙상, 등록시에는 최대값이 100까지만 가능하지만, 수정시에는 그러한 최대값 제한이 없을 수 있다.
		- 이 경우 `@Max(100)` 를 통해 Item 의 quantity 컬럼에 제약을 걸어버리면, 수정시에 최대값 제한 없이 수정할 수 있어야 한다는 요구사항은 지킬 수 없다.

- 이처럼 서로 다른 요구 사항 때문에 문제가 발생할 수 있다.

### 해결 방안
- 동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하는 방법
		1. BeanValidation의 groups 기능을 사용
		2. Item 객체를 직접 사용하는 대신, ItemSaveDto, ItemUpdateDto와 같이 별도의 모델 객체를 만들어서 사용

#### groups 사용
- Bean Validation이 제공하는 groups 기능을 사용하면, 상황에 따라 검증할 기능을 그룹으로 나누어 적용할 수 있다.

- groups 생성
```java
// 저장용 groups 생성
public interface SaveCheck {
}

// 수정용 groups 생성
public interface UpdateCheck {
}

```

- Item 클래스에 groups 적용
```java
@Data  
public class Item {  
  
    @NotNull(groups = UpdateCheck.class)  
    private Long id;  
  
    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})  
    private String itemName;  
  
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})  
    @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})  
    private Integer price;  
  
    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})  
    @Max(value = 9999, groups = {SaveCheck.class})  
    private Integer quantity;  
  
    public Item() {  
    }  
  
    public Item(String itemName, Integer price, Integer quantity) {  
        this.itemName = itemName;  
        this.price = price;  
        this.quantity = quantity;  
    }  
}
```

- [Controller](../Spring%20MVC/Controller.md) 로직에 Groups 적용
```java
// save group 적용
@PostMapping("/add")  
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
	...
}

// update group 적용
@PostMapping("/{itemId}/edit")  
public String editV2(@PathVariable Long itemId, @Validated(UpdateCheck.class) @ModelAttribute Item item, BindingResult bindingResult) {
	...
}
```

> [!info] groups는 @Validated에서만
> - @Valid는 groups 기능이 없다.
> - groups를 사용하고자 한다면 @Validated를 사용


#### 객체 분리 (권장)
- groups 방식은 실질적으로 잘 사용되지 않음
	- 요청시 전달 받는 데이터는 보통 Item 등의 도메인 객체와 딱 맞아떨어지지 않기 때문
		- 필요한 정보 뿐 아니라 약관 정보 등의 수많은 부가 정보들이 같이 전송되고
		- Item 객체에도 생성일자, 수정일자와 같은 수많은 부가 정보들이 존재
- 따라서 [Controller](../Spring%20MVC/Controller.md)에서 각 요청이 보내는 데이터에 꼭 맞는 별도의 객체를 만들어서, 데이터 전달 만을 위해 사용하는 것을 더 권장
- 이 경우 SaveDto와 EditDto를 별도로 만들고, 각 Dto에 맞게 BeanValidation을 적용할 수 있다.
- `request -> dto -> controller -> Item(dto를 기반으로 생성) -> service / repository`


## HTTP 메세지 컨버터 (@RequestBody, API)에 적용

- `@Valid`, `@Validated`는 [HttpMessageConverter](../Spring%20MVC/HttpMessageConverter.md) (@RequestBody 사용)에도 적용이 가능
```java
@PostMapping("/add")  
public Object addItem(@RequestBody @Validated ItemSaveForm form, BindingResult bindingResult) {  
  
    log.info("API 컨트롤러 호출");  
  
    if (bindingResult.hasErrors()) {  
        log.info("검증 오류 발생 errors={}", bindingResult);  
        return bindingResult.getAllErrors();  
    }  
  
    log.info("성공 로직 실행");  
    return form;  
}
```

- API의 경우 3가지 시나리오
	- 성공 요청 : 성공
	- 실패 요청 : JSON을 객체로 변환하는 것부터 실패
		- 이 경우에는 객체를 만들지도 못했기 때문에 컨트롤러가 아예 호출되지 않고, Validator도 실행되지 않는다.
	- 검증 오류 요청 : JSON으로 변환은 성공 했으나, 검증에서 실패
		- `return bindingResult.getAllErrors();`에서 ObjectError와 FieldError를 반환
		```json
		[  
			{
			
			"codes": [
				"Max.itemSaveForm.quantity",
				"Max.quantity",
				"Max.java.lang.Integer",
				"Max"
			
			],
			"arguments": [
			
				{  
					"codes": [
					
						"itemSaveForm.quantity",
					
					"quantity"
					
					],        
					"arguments": null,
					"defaultMessage": "quantity",
					"code": "quantity"
				
				},	
				9999
			],  
			"defaultMessage": "9999 이하여야 합니다",
			
			"objectName": "itemSaveForm",
			"field": "quantity",
			"rejectedValue": 10000,
			"bindingFailure": false,
			"code": "Max"
			
			} 
		]
		```
- 이 객체를 바로 반환하기 보다는, 필요한 데이터만 파싱해서 별도의 API 스펙에 맞춰 반환하는 것을 추천

> [!info] @ModelAttribute vs @RequestBody
> - @ModelAttribte의 경우 각각의 필드 단위로 세밀하게 적용
> 	- 특정 필드에 타입 오류가 발생해도 나머지 필드는 정상 처리
> - @RequestBody(HttpMessageConverter)의 경우에는 필드 단위가 아니라 전체 객체 단위로 적용
> 	- 일단 JSON을 메세지 컨버터를 통해 DTO로 변환하는데 성공해야 @Valid, @Validated 적용 가능
> 	- JSON 데이터를 객체로 변경하지 못하면 이후 단계로 진행하지 않고 예외 발생
> 	- 컨트롤러 호출도 되지 않고, Validator도 적용 X
