# Validator

```table-of-contents
```

##  Validator란?

- 스프링은 검증을 체계적으로 제공하기 위해 Validator 인터페이스를 제공
```java
 public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```
- supports : 해당 검증기를 지원하는지 여부를 확인해서 반환하는 메서드
- validate(Object target, Errors error) : 검증 대상 객체와 [BindingResult](BindingResult.md)를 넣어서 검증

## 예시 코드

### Validator 구현
```java

@Component  
public class ItemValidator implements Validator {  
    @Override  
    public boolean supports(Class<?> clazz) {  
        return Item.class.isAssignableFrom(clazz);  
    }  
  
    @Override  
    public void validate(Object target, Errors errors) {  
        Item item = (Item) target;  
        // 검증 로직  
  
        if (!StringUtils.hasText(item.getItemName())) {  
            errors.rejectValue("itemName", "required");  
        }  
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {  
            errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);  
        }  
        if (item.getQuantity() == null || item.getQuantity() > 9999) {  
            errors.rejectValue("quantity", "max", new Object[]{1000, 1000000}, null);  
  
        }  
  
        // 복합 룰 검증  
        if (item.getPrice() != null && item.getQuantity() != null) {  
            int resultPrice = item.getPrice() * item.getQuantity();  
            if (resultPrice < 10000) {  
                errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);  
            }  
        }  
    }  
}
```


## WebDataBinder
- 직접 [@Autowired](../@Autowired.md)를 통해 불러온 후에 사용하는 것도 가능하지만, WebDataBinder에 등록하게 되면, 자동으로 적용하는 것이 가능하다.

### WebDataBinder에 Validator 등록

#### 단일 컨트롤러 적용
```java
@InitBinder  
public void init(WebDataBinder dataBinder) {  
    dataBinder.addValidators(itemValidator);  
}

```
- `@InitBinder` 의 경우 해당 컨트롤러에만 영향을 줌

#### 글로벌 적용
```java
@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {

	public static void main(String[] args) {
		SpringApplication.run(ItemServiceApplication.class, args);

	}

	@Override      
	public Validator getValidator() {
		return new ItemValidator();
	}

}
```

> [!Important] BeanValidator 등록 문제
> 이처럼 글로벌 설정으로 등하는 경우에는 [BeanValidator](BeanValidator.md)가 자동 등록되지 않기 때문에 잘 사용하지 않는다.



### @Validated
- WebDataBinder에 등록되어 있는 검증기를 실행시키는 어노테이션
- 검증기들에 정의되어 있는 `supports(Item.class)`를 호출해서, 적합한 검증기를 찾고, 그 검증기의 `validate()`를 실행

```java
@PostMapping("/add")  
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {  
  
  
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

### @Valid
- `@Validated` 처럼 `@Valid`도 검증에 사용가능
- `@Valid`를 사용하기 위해서는 `build.gradle`에 의존관계 추가 필요
	- `implementation 'org.springframework.boot:spring-boot-starter-validation'` 
- `@Validated`는 스프링 전용 검증 어노테이션, `@Valid`는 자바 표준 검증 어노테이션
