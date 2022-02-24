> Controller (이하 서비스는 생략)

```java
    @GetMapping("/md")
    public void test(@ModelAttribute("text") String text) {
        //새 글 작성이나, 수정 시 나오는 화면
    }

    @GetMapping("/markdownAfterSave")
    public String test3(@RequestParam("text") String text) {
        //DB에 저장하고 list화면으로 간다
        service.save(text);        
        return "redirect:/list";        
    }

    @GetMapping("/list")
    public void list(Model model) {
        //목록
        model.addAttribute("list", service.getAll());
    }
    
    
    @GetMapping("/read")
    public void read(@ModelAttribute("text") String text) {
    }
```

<br/>

> DAO

```java
    @Insert("insert into markdown(text) values(#{text})")
    public void save(String text);

    @Select("select text from markdown")
    public List<String> getAll();
```

<br/>

> VIEW

- 목록보기(list)

```jsp
    <!-- 목록들 출력 -->
	<c:forEach items="${list}" var="obj">
		<p><c:out value="${obj}"></c:out></p>
	</c:forEach>
	
	<button id="write">글쓰기</button>

	<script>
        //글쓰기 버튼 동작
		document.querySelector("#write").addEventListener("click",()=>{location.href = "/md";});
		
        //목록에서 각 게시글들을 누르면 상세보기로 간다.
		const ps = document.querySelectorAll("p");
		for(var i=0;i<ps.length;i++){
			ps[i].addEventListener("click",(e) => {location.href = '/read?text='+encodeURIComponent(e.target.innerText);});
		}
	</script>
```

- 작성/수정(md)

```jsp
	<textarea id = 'before'>${text}</textarea><!-- 텍스트를 입력할 상자 -->
	<div id = 'after' style=""></div><!-- 렌더링 된 html을 보여줄 영역 -->	
	<button id="save">저장하기</button>
	
	<!-- markdown형식의 plain-text를 html로 rendering 해주기 위한 라이브러리 -->
	<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>	
	<script type="text/javascript">
	
		const before = document.querySelector("#before");//텍스트를 입력할 상자
		const after = document.querySelector("#after");//렌더링 된 html을 보여줄 영역
		const save = document.querySelector("#save");//버튼
		
		after.innerHTML = marked.parse(before.value);//초기 렌더링
		
        //텍스트 상자가 입력될 때마다 렌더링
		before.addEventListener("input",() => {after.innerHTML = marked.parse(before.value);});
        //저장버튼 누르면 저장
		save.addEventListener("click",() => {location.href = '/markdownAfterSave?text='+encodeURIComponent(before.value);});
				
	</script>
```

- 상세보기(read)

```jsp
	<div></div><!-- 렌더링 된 화면을 보여줘야함 -->
	<button>수정하기</button>
    
    <!--라이브러리-->
	<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
	<script type="text/javascript">
    	//화면에 원본 데이터를 렌더링 하여 보여줌
		document.querySelector("div").innerHTML = marked.parse('${text}');
        
        //수정하기 버튼 누르면 그 내용을 가지고 작성/수정으로 이동
		document.querySelector("button").addEventListener("click",()=>{
			location.href = '/md?text='+encodeURIComponent('${text}');;
		});
	</script>
```

<br/>

> 결과 화면

1. 목록(list)

![](https://images.velog.io/images/sonchanwoo/post/09ac1766-1a6b-4ef0-9b6c-38e55100c810/image.png)

2. 작성(md) : 실시간 렌더링 됨

![](https://images.velog.io/images/sonchanwoo/post/998f1f4a-a717-4d88-82c9-c7f5dea31042/image.png)

3. 저장 후 목록

![](https://images.velog.io/images/sonchanwoo/post/d8b3c5d5-d98b-4615-9cac-4dd9c6218a99/image.png)

4. 상세보기(read) : **개행문자가 씹히는 현상이 발생함!!**

![](https://images.velog.io/images/sonchanwoo/post/90642290-a8d8-4ea9-9b19-c4bdf6b491c4/image.png)

5. 수정(md) : **등록 때와 비교!!**

![](https://images.velog.io/images/sonchanwoo/post/54dec4fc-8aec-4717-a8ff-433389dd8a48/image.png)

<br/>

- 등록 시에 엔터로 구분을 해줘도 저장/수정 후에는 엔터가 없이 들어간다

- 현재 수정이 안되는 건 급하게 대충 만든다고 그런 것

-> 다음 포스팅은 수정 고치고, 엔터 및 그 밖의 문제를 모두 해결하는 내용 예정
