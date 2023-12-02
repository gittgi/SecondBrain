# Formatter

```table-of-contents
```
> [!Important] **메세지 컨버터에는 컨버전 서비스 적용 불가**
> - [HttpMessageConverter](Spring%20MVC/HttpMessageConverter.md)에는 컨버전 서비스가 적용되지 않음
> 	- 메세지 컨버터는 HTTP body의 내용을 객체로 변환 또는 객체를 body에 입력하는 것
> 	- 이 변환에는 메세지 컨버터 내부에서 쓰는 [Jackson](../미완성%20문서/Jackson.md)라이브러리가 사용됨
> 	- 즉, JSON 결과로 만들어지는 숫자나 날짜 포맷을 변경하고 싶은 경우에는 해당 라이브러리의 설정을 통해 지정해야 함
> - 결과적으로 Formatter 및 [ConversionService](스프링%20타입%20컨버터.md)는 `@RequestParam`, `@ModelAttribue`, `@PathVariable`, 뷰 템플릿 등에서 사용
##  Formatter란?

- [스프링 타입 컨버터](스프링%20타입%20컨버터.md)의 문자 특화 버전
- Converter의 경우에는 입력과 출력 타입에 제한이 없는, 범용 타입 변환 기능을 제공
- 그러나 일반적인 웹 어플리케이션 환경에서는 문자를 다른 타입의 객체로 변환하거나, 객체를 문자로 변환하는 것이 대부분
- 이처럼 객체를 특정한 포맷의 문자로 출력하거나 그 반대를 수행하는 것에 특화된 기능을 제공하는 것이 Formatter

## Formatter 인터페이스

```java
public interface Printer<T> {
	String print(T object, Locale locale);

}

public interface Parser<T> {
	T parse(String text, Locale locale) throws ParseException;

}

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```
- Formatter 인터페이스는 두가지 인터페이스(Printer, Parser)를 구현
	- `String print(T object, Locale locale)` : 객체를 문자로 변경
	- `T parse(String text, Locale locale)` : 문자를 객체로 변경


## Formatter 구현 예시
```java
@Slf4j  
public class MyNumberFormatter implements Formatter<Number> {  
  
    @Override  
    public Number parse(String text, Locale locale) throws ParseException {  
        log.info("text={}, locale={}", text, locale);  
        NumberFormat format = NumberFormat.getInstance(locale);  
        return format.parse(text);  
    }  
    @Override  
    public String print(Number object, Locale locale) {  
        log.info("object={}, locale={}", object, locale);  
        return NumberFormat.getInstance(locale).format(object);  
    }  
  
}
```
- 자바가 기본으로 제공하는 `NumberFormat`객체를 사용하면 숫자 세자리마다 쉼표 가능, 또한 Locale정보를 활용해서 나라마다 숫자 포맷을 맞춰줌
- `parse()`는 문자를 객체로, `print()`는 객체를 문자로 바꿔준다.


## Formatter 등록

- [ConversionService](스프링%20타입%20컨버터.md) 컨버터만 등록할 수 있기 때문에 대신 FormattingConversionService를 통해 등록해야 한다.
	- [어댑터 패턴](Spring%20MVC/어댑터%20패턴.md)을 적용하여 Formatter도 Converter처럼 동작하게 했기 때문에, 컨버터도 등록하고 포매터도 등록할 수 있다.


- DefaultFormattingConversionService는 FormattingConversionService에 기본적인 통화, 숫자 관련 포맷터를 추가해 둔 것
```java
@Test  
void formattingConversionService() {  
    DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();  
  
    conversionService.addConverter(new IpPortToStringConverter());  
    conversionService.addConverter(new StringToIpPortConverter());  
  
    conversionService.addFormatter(new MyNumberFormatter());  
}
```

### DefaultFormattingConversionService의 상속관계
- FormattingConversionService는 ConversionService 관련 기능을 상속
	- 컨버터 & 포매터 모두 상속 가능
	- 사용 인터페이스인 ConversionService가 제공하는 convert로 사용
- [스프링부트](../미완성%20문서/SpringBoot.md)는 DefaultFormattingConversionService를 상속 받는 WebConversionService를 내부적으로 사용

## Formatter 적용

- 어플리케이션에 적용하기 위해서는 [스프링 타입 컨버터](스프링%20타입%20컨버터.md) 를 등록했던 것처럼, [WebMvcConfigurer](../미완성%20문서/WebMvcConfigurer.md) 에 등록

```java
@Configuration  
public class WebConfig implements WebMvcConfigurer {  
    @Override  
    public void addFormatters(FormatterRegistry registry) {  
        registry.addConverter(new StringToIpPortConverter());  
        registry.addConverter(new IpPortToStringConverter());  
  
        registry.addFormatter(new MyNumberFormatter());  
    }  
}
```
- [스프링 타입 컨버터](스프링%20타입%20컨버터.md) 때와 마찬가지로 addFormatter를 오버라이드해서 추가해 줄 수 있는데, Formatter의 경우에는 addFormatter()로 등록
- 기능이 겹치는 경우 (숫자 -> 문자 등) 컨버터가 포맷터보다 우선적으로 적용된다.


## 스프링이 제공하는 기본 포맷터

- 스프링은 용도에 따라 다양한 방식의 포맷터 제공
	- Formatter : 포맷터
	- AnnotationFormatterFactory : 필드의 타입이나 어노테이션 정보를 활용할 수 있는 포맷터
	- 자세한 내용은 https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format 참조

- 스프링은 날짜나 시간 관련 포맷터를 다수 제공
- 그러나 각 필드마다 다른 형식을 제공하기 위해 어노테이션 기반 포맷터를 제공
	- `@NumberFormatter` : 숫자 관련 형식 지정 포맷터 사용 (NumberFormatAnnotationFormatterFactory)
	- `@DateTimeFormat` : 날짜 관련 형식 지정 포맷터 사용 (Jsr310DateTimeFormatAnnotationFormatterFactory)

### 예시

```java
@Controller

  public class FormatterController {

      @GetMapping("/formatter/edit")
      public String formatterForm(Model model) {
	      Form form = new Form();
          form.setNumber(10000);
          form.setLocalDateTime(LocalDateTime.now());

          model.addAttribute("form", form);

          return "formatter-form";
      }

      @PostMapping("/formatter/edit")
      public String formatterEdit(@ModelAttribute Form form) {
          return "formatter-view";
      }

      @Data      
      static class Form {
          @NumberFormat(pattern = "###,###")
          private Integer number;

          @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
          private LocalDateTime localDateTime;
      }

}
```

- 자세한 사용방식은 https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format-CustomFormatAnnotations 참조