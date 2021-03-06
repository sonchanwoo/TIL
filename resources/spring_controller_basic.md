
> ๐ controller์ ๋น ๋ฑ๋ก

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

1. ํ๋ก์ ํธ ์์ฑ ์ base package๋ servlet-context์ ์๋์ผ๋ก ์ค์บ๋๋ค.

2. root-context์ฒ๋ผ ์ค์บ+์ด๋ธํ์ด์์ผ๋ก ์ธํ์ฌ ๋น๋ฑ๋ก์ด ์๋ฃ๋๋ค.

<br/>

> ๐ Mapping ์คํ

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

1. log.info์ ๋ด๋ถ๋ ์ฃผ์์ฐฝ์ ์ฐ์ด์ผ ํ  ๊ฒฝ๋ก์ด๋ค.
2. ์์ ๋งคํ์ ๋น๊ตํ๋ฉด์ ๋ณผ ๊ฒ
3. ์ 3๊ฐ๋ ๋์ผํ๋ค. (๊ธฐ๋ณธ์ ์ผ๋ก ์ธ๊ธ์ด ์์ผ๋ฉด ๋ชจ๋  ๋ฐฉ์์ผ๋ก ๋์ํ๋ค.)

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

1. ์ ๋๊ฐ๋ ๊ฐ๋ค.

2. get๋ฐฉ์์ด ์๋ ๋ฐฉ์์ผ๋ก ์์ฒญ์ด ๋ค์ด์ค๋ฉด method not allowed๋ผ๋ 405์๋ฌ๋ฅผ ๋ฐ์์ํจ๋ค.

3. ๊ฐ์ ๋งฅ๋ฝ์ผ๋ก PostMapping๋ ๋์ผํ๋ค.

<br/>

> ๐ ํ๋ผ๋ฏธํฐ ์์ง

- DTO ์ค๋น

```java
@Data
public class SampleDTO {
    private String name;
    private int age;
}
```

<br/>

- ์๋ ฅํ  ๊ฒฝ๋ก : localhost:8080/sample/ex01?name=AAA&age=10

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

1. ์๋ธ๋ฆฟ์์ request.getParameter()๋ด์ฉ์ ํด๋นํ๋ ๋ถ๋ถ์ด๋ค.

2. ๋งคํ๋ ๋ฉ์๋์ ํ๋ผ๋ฏธํฐ์ ์ ์ธ๋ง ํด์ฃผ์ด๋ ์๋์ผ๋ก ์์ง๋๋ค.

3. ์ด๋ฆ์ด ๋ค๋ฅธ ๊ฒฝ์ฐ์ @RequestParam๊ฐ ์ ์ฉํ๋ค.

4. ๊ฐ์ ๊ฒฝ์ฐ์๋ ์ด๋ธํ์ด์์ ์๋ตํ  ์ ์๋ค.

5. ๊ฐ์ฒดํ์์ผ๋ก ๋ฐ์ ์๋ ์์์ ์ฃผ๋ชฉํ๋ค.(ํ๋กํผํฐ)

6. ๋ฆฌ์คํธ(๋ฐฐ์ด)๋ก๋ ๋ฐ์ ์ ์๋ค.(์ฌ์คํ๊ฒ ํฌ์คํ ํ  ์์ )

<br/>

> ๐ ๋ฉ์๋์ returnํ์

- void : ํด๋น ๋ฉ์๋๋ฅผ URI ๊ฒฝ๋ก์ ๊ฐ์ jspํ์ผ์ ์ฐพ๋๋ค.
  - /sample/ex01 -> (/WEB-INF/views) /sample/ex01.jsp

- String : ์ง์  ์ง์ ํ  ๋ ์ฌ์ฉํ๋ค
  - return "ex01" -> (/WEB-INF/views) /ex01.jsp

<br/>

> ๐ ๋ฐ์ดํฐ ์ ๋ฌ

- ์๋ ฅํ  ๊ฒฝ๋ก : localhost:8080/sample/ex04?name=chanwoo&age=24&page=9

- Model

```java
    @GetMapping("/ex02")
    public void ex02(SampleDTO dto, int page, Model model) {  
        model.addAttribute("page", page);
    }
```

1. ์ปจํธ๋กค๋ฌ์์ view๋ก ๋์ด๊ฐ ๋, ๋ฐ์ดํฐ๋ฅผ ์ ๋ฌํ๊ธฐ๋ฅผ ์ํ๋ค.

2. Model ๊ฐ์ฒด๊ฐ ์๋ธ๋ฆฟ์ req.setAttribute ์ญํ ์ ํด์ค๋ค.

3. dto๋ ์๋ฐ๋น์ฆ๊ท์น์ ๋ง์กฑํ๊ธฐ ๋๋ฌธ์ ์๋์ผ๋ก view๋ก ์ ๋ฌ์ด ๋๋ค.(ํด๋์ค ์ด๋ฆ+์ฒซ๊ธ์ ์๋ฌธ์๋ก ๊ฐ๋ค.)

4. ๊ธฐ๋ณธ ์๋ฃํ ํ์์ ๋ณ์์ธ page๋ ์๋ฐ๋น์ฆ๊ฐ ์๋๊ธฐ ๋๋ฌธ์ model.addAttribute()๋ฅผ ์ด์ฉํจ์ ๋ณผ ์ ์๋ค.

5. ์ฌ์ค ํ๋ผ๋ฏธํฐ๋ฅผ ์ฆ์ ์ ๋ฌํ๋ ๋ฐฉ๋ฒ์ ์ด๋ธํ์ด์์ ์ด์ฉํ  ์ ์๋ค.

```java
    @GetMapping("/ex03")
    public void ex03(SampleDTO dto, @ModelAttribute("page") int page, Model model) { 
    }
```

6. Model๊ฐ์ฒด๋ ํ๋ผ๋ฏธํฐ๊ฐ ์๋ ๊ฒ์ด๋ ํ๋ผ๋ฏธํฐ๋ฅผ ๊ฐ๊ณตํ ๊ฒ์ ๋ณด๋ผ ๋ ์ฐ์ธ๋ค.(page๋ก ๋ถ๋ฌ์จ ๋ชฉ๋ก ๋ฆฌ์คํธ ๋ฑ)

<br/>

> ๐ Redirect

```java
    @GetMapping("/ex04")
    public String ex04(RedirectAttributes redirectAttributes) {
        redirectAttributes.addFlashAttribute("age",9);

        return "redirect:/";
    }

```

1. redirect๋ ๋ฐํ๊ฐ์ redirect: ํค์๋๋ฅผ ์ฌ์ฉํ๋ค.

2. rttr.addFlashAttribute()๋ ์ผํ์ฑ ๋ฐ์ดํฐ๋ฅผ ์๋ฏธํ๋ค.

