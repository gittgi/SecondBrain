# BindingResult

```table-of-contents
```

##  BindingResult란?

- 스프링에서 제공하는 검증 오류 보관 객체
- [Controller](../Spring%20MVC/Controller.md)에 들어오는 데이터에 바인딩 오류가 발생하면, 보통은 400 오류 발생과 함께 컨트롤러가 호출되지 않고 오류 페이지로 돌아감
- 그러나 BindingResult가 있으면 바인딩 오류 정보가 BindingResult에 담고 **컨트롤러를 정상 호출**함
	- BindingResult는 [Model](../../미완성%20문서/Model.md)로 자동 포함됨
- BindingResult에 검증 오류를 적용하는 방법
	- [@ModelAttribute](../Spring%20MVC/Controller.md)의 객체 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 FieldError를 생성해서 BindingResult에 넣어줌
	- 개발자가 코드로 직접 넣어줌
	- [Validator](Validator.md)사용

> [!important] BindingResult 위치 주의
> BindingResult의 파라미터 위치는 항상 **검증할 대상 바로 뒤**에 와야 함 


### BindingResult 인터페이스 

- BindingResult는 Errors 인터페이스를 상속받은 인터페이스
	- `org.springframework.validation.Errors`
	- `org.springframework.validation.BindingResult`
- 실제 파라미터로 넘어오는 구현체는 BeanPropertyBindingResult
- BindingResult 대신에 Errors를 쓸수도 있으나, addError() 등의 기능은 BindingResult가 제공하는 추가 기능이기 때문에 주로 BindingResult 사용

## FieldError, ObjectError

### 예시 코드

```java
@PostMapping("/add")  
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {  
  
	// 검증 로직  
	if (!StringUtils.hasText(item.getItemName())) {  
		bindingResult.addError((new FieldError("item", "itemName", item.getItemName(), false, null, null, "상품 이름은 필수입니다.")));  
	}  
	if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {  
		bindingResult.addError((new FieldError("item", "price", item.getPrice(), false, null, null, "가격은 1,000 ~ 1,000,000 까지만 허용합니다.")));  
	}  
	if (item.getQuantity() == null || item.getQuantity() > 9999) {  
		bindingResult.addError((new FieldError("item", "quantity", item.getQuantity(), false, null, null, "수량은 최대 9,999 까지 허용합니다.")));  
	}  

	// 복합 룰 검증  
	if (item.getPrice() != null && item.getQuantity() != null) {  
		int resultPrice = item.getPrice() * item.getQuantity();  
		if (resultPrice < 10000) {  
			bindingResult.addError(new ObjectError("item", null, null, "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));  
		}  
	}  

	// 검증 실패시 다시 입력 폼으로  
	if (bindingResult.hasErrors()) {  
		log.info("errors = {}", bindingResult);  

		return "validation/v2/addForm";  
	}  

	// 성공 로직  

	Item savedItem = itemRepository.save(item);  
	redirectAttributes.addAttribute("itemId", savedItem.getId());  
	redirectAttributes.addAttribute("status", true);  
	return "redirect:/validation/v2/items/{itemId}";  
    }
```


### FieldError
- **단일 필드에 오류**가 있으면 객체를 생성해서 bindingResult에 담음
#### FieldError 생성자
```java
public FieldError(String objectName, String field, String defaultMessage);

public FieldError(String objectName, String field, @Nullable Object rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
```
- objectName : 오류가 발생한 객체 이름
- field : 오류 필드
- rejectedValue : 사용자가 입력한 값(거절된 값)
- bindingFailure : 타입 오류 등의 바인딩 실패(true)인지, 검증 실패(false)인지 구분하는 값
- codes : 메세지 코드
- arguments : 메세지에서 사용하는 인자
- defaultMessage : 기본 오류 메세지


### ObjectError
- **단일 필드를 넘어서는 오류**의 경우에는 ObjectError 객체를 만들어서 bindingResult에 담음

#### ObjectError 생성자
```java
public ObjectError(String objectName, String defaultMessage)

public ObjectError(String objectName, @Nullable String[] codes, @Nullable Object[] arguments, @Nullable String defaultMessage)
```
- objectName : 오류가 발생한 객체 이름
- codes : 메세지 코드
- arguments : 메세지에서 사용하는 인자
- defaultMessage : 기본 오류 메세지

### codes, arguments로 에러 메세지 지정
- FieldError와 ObjectError의 생성자는 codes, arguments 값을 받음
- 이는 오류 발생시 오류 코드로 메세지를 찾을 때 사용
- 방법
	- application.properties에 스프링 부트 메세지 설정 추가
		- `spring.messages.basename=messages,errors`
	- errors.properties 추가
		- `range.item.price=가격은 {0} ~ {1} 까지 허용합니다.`
	- Error 생성시 codes와 arguments 넣기
		```java
		//range.item.price=가격은 {0} ~ {1} 까지 허용합니다.  
		new FieldError("item", "price", item.getPrice(), false, new String[] {"range.item.price"}, new Object[]{1000, 1000000}
		```
		- codes : range.item.price을 사용해서 메세지 코드 지정, 메세지 코드는 배열로 여러값을 지정해서, 순서대로 매칭해보고 처음 매칭되는 메세지를 사용
		- arguments : 메세지의 {0}, {1} 을 치환할 값을 넣을 수 있음
		- 자세한건 [MessageSource](../../미완성%20문서/MessageSource.md) 참조


## rejectValue(), reject()

- 매번 FieldError나 ObjectError를 정의하여 생성하는 것은 불편
- BindingResult가 제공하는 rejectValue(), reject()를 사용하면 좀 더 쉽게 오류 메세지 출력 가능

### rejectValue()

```java
// bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null)
void rejectValue(@Nullable String field, String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);
```
- field : 오류 발생 필드명
- errorCode : 오류 코드 ([MessageCodesResolver](MessageCodesResolver.md)를 위한 오류 코드)
- errorArgs : 오류 메세지에서 {0} 등을 치환하기 위한 값
- defaultMessage : 오류 메세지를 찾을 수 없을 때 사용하는 기본 메세지

### reject() 
- reject()는 객체 오류 용
```java
void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String
  defaultMessage);
```

> [!info] Target에 대한 정보 
> BindingResult은 검증할 객체 바로 뒤에 붙어서, 검증 대상에 대해 알고 있기 때문에, target에 대한 정보는 불필요

### ValidationUtils
- Empty나 공백 같은 단순한 기능에 한해서, 좀 더 간편하게 rejectValue(), reject()를 사용할 수 있게 해주는 객체

```java
// 기존
if (!StringUtils.hasText(item.getItemName())) { 
	bindingResult.rejectValue("itemName", "required", "기본: 상품 이름은 필수입니다."); 
}
	
// ValidationUtils
ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, "itemName", "required");

```
- rejectValue() 호출
- [MessageCodesResolver](MessageCodesResolver.md)를 사용해서 메세지 코드들 생성
- `new FieldError()`를 생성하면서 메세지 코드들 보관


## 스프링이 만든 오류 코드
- 개발자가 직접 설정한 오류 코드의 경우 rejectValue()를 통해서 직접 호출이 가능
- 스프링 역시 직접 검증 오류에 추가한 경우가 있다. (주로 타입 오류)

### 예시 
- 스프링의 경우, 타입 오류가 발생하면 typeMismatch라는 오류 코드를 사용, [MessageCodesResolver](MessageCodesResolver.md)를 통해 4가지 메세지 코드를 생성
- 따라서 개발자는 `error.properties`에 해당 하는 오류 메세지를 정의해두면 스프링이 검증해주는 오류에 대응해서도 메세지를 쓸 수 있다.
```properties
typeMismatch.java.lang.Integer=숫자를 입력해주세요.
typeMismatch=타입 오류입니다.
```


## 검증 로직 분리

- 검증로직을 분리하기 위해서 스프링은 별도의 인터페이스를 제공한다.
- 자세한 내용은 [Validator](Validator.md) 참조