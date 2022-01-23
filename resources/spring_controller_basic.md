
> 🚀 controller의 빈 등록

- @Controller

```java
@Controller
@RequestMapping("/sample/*")
@Log4j
public class SampleController {
    
    @RequestMapping(value="/")
    public void basicGet() {
        log.info("/sample/");
    }
}
```

- servlet-context.xml

```xml
<context:component-scan base-package="org.zerock.controller" />
```

1. 프로젝트 생성 시 base package는 servlet-context에 자동으로 스캔된다.

2. root-context처럼 스캔+어노테이션으로 인하여 빈등록이 완료된다.

<br/>

> 🚀 Mapping 실험

```java
    @RequestMapping("/")
    public void basic() {
        log.info("localhost:8080/sample/");
    }

    @RequestMapping(value="/")
    public void basic() {
        log.info("localhost:8080/sample/");
    }

    @RequestMapping(value="/", method={RequestMethod.GET, RequestMethod.POST, ...})
    public void basic() {
        log.info("localhost:8080/sample/");
    }    
```

1. log.info의 내부는 주소창에 찍어야 할 경로이다.
2. 위와 매핑을 비교하면서 볼 것
3. 위 3개는 동일하다. (기본적으로 언급이 없으면 모든 방식으로 동작한다.)

<br/>

```java
    @RequestMapping(value="/get",method=RequestMethod.GET)
    public void get() {
        log.info("localhost:8080/sample/get");
    }
    
    @GetMapping("/get")
    public void get() {
        log.info("localhost:8080/sample/get");
    }
```

1. 위 두개는 같다.

2. get방식이 아닌 방식으로 요청이 들어오면 method not allowed라는 405에러를 발생시킨다.

3. 같은 맥락으로 PostMapping도 동일하다.

<br/>

> 🚀 파라미터 수집

- DTO 준비

```java
@Data
public class SampleDTO {
    private String name;
    private int age;
}
```

<br/>

- 입력할 경로 : localhost:8080/sample/ex01?name=AAA&age=10

<br/>

- controller - 1

```java
    @GetMapping("/ex01")
    public void ex01(@RequestParam("name") String name, @RequestParam("age") int age) {
    }

    @GetMapping("/ex01")
    public void ex01(String name, int age) {
    }

    @GetMapping("/ex01")
    public void ex01(SampleDTO dto) {
    }
```

1. 서블릿에서 request.getParameter()내용에 해당하는 부분이다.

2. 매핑된 메서드의 파라미터에 선언만 해주어도 자동으로 수집된다.

3. 이름이 다른 경우에 @RequestParam가 유용하다.

4. 같은 경우에는 어노테이션을 생략할 수 있다.

5. 객체타입으로 받을 수도 있음에 주목한다.(프로퍼티)

6. 리스트(배열)로도 받을 수 있다.(심오하게 포스팅 할 예정)

<br/>

> 🚀 메서드의 return타입

- void : 해당 메서드를 URI 경로와 같은 jsp파일을 찾는다.
  - /sample/ex01 -> (/WEB-INF/views) /sample/ex01.jsp

- String : 직접 지정할 때 사용한다
  - return "ex01" -> (/WEB-INF/views) /ex01.jsp

<br/>

> 🚀 데이터 전달

- 입력할 경로 : localhost:8080/sample/ex04?name=chanwoo&age=24&page=9

- Model

```java
    @GetMapping("/ex02")
    public void ex02(SampleDTO dto, int page, Model model) {  
        model.addAttribute("page", page);
    }
```

1. 컨트롤러에서 view로 넘어갈 때, 데이터를 전달하기를 원한다.

2. Model 객체가 서블릿의 req.setAttribute 역할을 해준다.

3. dto는 자바빈즈규칙을 만족하기 때문에 자동으로 view로 전달이 된다.(클래스 이름+첫글자 소문자로 간다.)

4. 기본 자료형 타입의 변수인 page는 자바빈즈가 아니기 때문에 model.addAttribute()를 이용함을 볼 수 있다.

5. 사실 파라미터를 즉시 전달하는 방법은 어노테이션을 이용할 수 있다.

```java
    @GetMapping("/ex03")
    public void ex03(SampleDTO dto, @ModelAttribute("page") int page, Model model) { 
    }
```

6. Model객체는 파라미터가 아닌 것이나 파라미터를 가공한 것을 보낼 때 쓰인다.(page로 불러온 목록 리스트 등)

<br/>

> 🚀 Redirect

```java
    @GetMapping("/ex04")
    public String ex04(RedirectAttributes redirectAttributes) {
        redirectAttributes.addFlashAttribute("age",9);

        return "redirect:/";
    }

```

1. redirect는 반환값에 redirect: 키워드를 사용한다.

2. rttr.addFlashAttribute()는 일회성 데이터를 의미한다.

