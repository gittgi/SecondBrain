---
tags:
  - MVC
  - Spring
---
# @RequestMapping

```table-of-contents
```

##  @RequestMapping란?

- [스프링 MVC](스프링%20MVC.md)에서 사용하는 어노테이션 기반 컨트롤러를 만들 때 사용하는 어노테이션


## 핸들러 매핑 및 핸들러 어댑터
- @RequestMapping을 이용한 어노테이션 기반 컨트롤러의 경우 다음의 핸들러 매핑 및 핸들러 어댑터와 연관된다. (가장 우선순위가 높은 핸들러 매핑과 어댑터이기도 하다.)
	- `RequestMappingHandlerMapping`
	- `RequestMappingHandlerAdapter`


## 실제 사용 예시

### 예시 코드

```java

@Controller  
@RequestMapping("/springmvc/v3/members")  
public class SpringMemberControllerV3 {  
  
    private final MemberRepository memberRepository = MemberRepository.getInstance();  
  
  
    @GetMapping("/new-form")  
    public String newForm() {  
        return "new-form";  
    }  
  
    @PostMapping("/save")  
    public String save(  
            @RequestParam("username") String username,  
            @RequestParam("age") int age,  
            Model model  
    ) {  
  
  
        Member member = new Member(username, age);  
        memberRepository.save(member);  
  
        model.addAttribute("member", member);  
        return "save-result";  
  
    }  
  
  
    @GetMapping  
    public String members(Model model) {  
        List<Member> members = memberRepository.findAll();  
        model.addAttribute("members", members);  
  
        return "members";  
  
    }  
  
}
```


### 예시 코드 분석

#### @Controller
- [@Controller](../../미완성%20문서/Controller.md) 어노테이션의 경우 내부에 [@Component](../../미완성%20문서/@Component.md) 어노테이션을 포함하고 있어서 [@ComponentScan](../@ComponentScan.md)의 대상이 됨
- [스프링 MVC](스프링%20MVC.md) 에서는 어노테이션 기반 컨트롤러로 인식

> [!important]
> [스프링 부트](../../미완성%20문서/SpringBoot.md) 3.0 (스프링은 6.0) 이상에서는 클래스 레벨에 `@RequestMapping`이 있어도 스프링 컨트롤러로 인식하지 않음 -> @Controller가 붙어 있어야 스프링 컨트롤러로 인식

#### @RequestMapping("URL")
- 요청 정보를 매핑
- 해당 URL로 요청이 오면 이 메서드가 호출됨
- 클래스 레벨과 메서드 레벨에 각각 붙여줄 수 있음
	- 두 레벨에 붙은 URL의 조합으로 요청 정보를 매핑함
	- 클래스 레벨 `@RequestMapping("/springmvc/v3/members") ` + 메서드 레벨 `@PostMapping("/save")` -> `/springmvc/v3/members/save`
- 어노테이션 기반이기 때문에, 메서드명의 경우에는 상관없이 자유롭게 지을 수 있음
##### @GetMapping, @PostMapping
- @RequestMapping에서 [HTTP method](../../CS/Web/HTTP%20method.md) 까지 함께 구분해서 매핑하는 어노테이션
- URL과 HTTP method까지 모두 일치하는 요청이 매핑됨
	- 같은 URL에 HTTP method별로 메서드를 지정하는 것도 가능
- GET외에도 HTTP 메서드 별로 어노테이션이 있음
- `@RequestMapping(method = RequestMethod.GET)`과 동일

#### 유연한 파라미터
- 어노테이션 기반의 컨트롤러의 경우 기존의 컨트롤러 처럼 [HttpServletRequest](HttpServletRequest.md)나 [HttpServletResponse](HttpServletResponse.md)를 직접 받을 수도 있지만, request에 포함된 파라미터 혹은 response에 전달할 데이터(모델)등만 직접 파라미터로 받는 것이 가능
##### Model 파라미터
- 스프링 MVC는 메서드의 파라미터로 [Model](../../미완성%20문서/Model.md)을 제공
- 이 Model을 통해 `addAttribute()` 등을 실행하고 자동으로 response에 담겨 View로 전해짐
##### @RequestParam
- 스프링은 [HTTP 요청](HttpServletRequest.md) 파라미터를 `@RequestParam`으로 간편하게 받을 수 있다.
	- `@RequestParam("username")` = `request.getParameter("username")`
	- GET 방식의 쿼리 파라미터와 POST Form 방식의 파라미터 모두에 적용

#### return 뷰 이름;
- 어노테이션 기반의 컨트롤러는 ModelAndView(`org.springframework.web.servlet.ModelAndView`)를 직접 반환해도 되지만, String으로 View의 논리 이름만 반환해도 된다. (유연한 설계)
	- 논리 이름만 반환시, 알아서 뷰의 이름으로 판단하고 해당 뷰를 렌더링한다.