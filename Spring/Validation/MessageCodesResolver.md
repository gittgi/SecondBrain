# MessageCodesResolver

```table-of-contents
```

##  MessageCodesResolver란?

- [BindingResult](BindingResult.md)에서 오류 메세지를 지정할 때, 오류 메세지의 단계를 두는 것으로 범용적인 메세지부터 세밀하고 정확한 메세지가 나눠서 출력할 수 있게 하는 방법
	- 범용 메세지 : `required: 필수 입니다.`
	- 상세 메세지 (우선 적용) : `required.item.itemName: 상품 이름은 필수 입니다.`
- 이렇게 계층을 나누는 것으로, 크게 중요하지 않은 메세지는 범용적으로 사용하고, 꼭 필요한 메세지는 구체적으로 적어서 사용할 수 있도록 하는 것이 관리하기 편하기 때문에 사용
- MessageCodesResolver는 인터페이스이고, DefaultMessageCodesResolver는 기본 구현체

## 동작 방식

- [BindingResult](BindingResult.md)의 `rejectValue()`, `reject()`는 내부에서 MessageCodesResolver를 사용
- FieldError, ObjectError의 생성자는 복수의 오류 코드를 가질 수 있는데, rejectValue(), reject()는 주어진 필드, 에러코드, 타입을 기반으로 2개(ObjectError) 또는 4개(FieldError)의 오류코드를 자동 생성해서 각 에러의 오류코드로 넣어준다.
	- 예시 
		- rejectValue("itemName", "required") -> FieldError의 codes : {"required.item.itemName", "required.itemName", "required.java.lang.String", "required"}



## DefaultMessageCodesResolver 기본 메세지 생성 규칙

구체적인 것 부터 매칭한다.

### 객체 오류
- 객체 오류의 경우 2가지 가능
	1. code + "." + object name (required.item)
	2. code (required)

### 필드 오류
- 필드 오류의 경우 4가지 가능
	1. code + "." + object name + "." + field ("typeMisMatch.user.age")
	2. code + "." + field ("typeMisMatch.age")
	3. code + "." + field type ("typeMisMatch.int")
	4. code ("typeMisMatch")



### errors.properties 예시
```properties

#==FieldError==  
#Level1  
required.item.itemName=상품 이름은 필수입니다. 
range.item.price=가격은 {0} ~ {1} 까지 허용합니다. 
max.item.quantity=수량은 최대 {0} 까지 허용합니다.

#Level2 생략

#Level3  
required.java.lang.String = 필수 문자입니다.  
required.java.lang.Integer = 필수 숫자입니다.  
min.java.lang.String = {0} 이상의 문자를 입력해주세요.  
min.java.lang.Integer = {0} 이상의 숫자를 입력해주세요.  
range.java.lang.String = {0} ~ {1} 까지의 문자를 입력해주세요.  
range.java.lang.Integer = {0} ~ {1} 까지의 숫자를 입력해주세요.  
max.java.lang.String = {0} 까지의 숫자를 허용합니다.  
max.java.lang.Integer = {0} 까지의 숫자를 허용합니다.  
  
#Level4  
required = 필수 값 입니다.  
min= {0} 이상이어야 합니다.  
range= {0} ~ {1} 범위를 허용합니다.  
max= {0} 까지 허용합니다.  

```