   지난 포스팅에서 LF(EOL, newline..등)가 씹히는 현상이 생겼었다.
   막연하게 인(디)코딩에서 문제가 있었을 거라 생각을 했다.

   log.info()를 해보니 컨트롤러 까지는 잘 들어왔다.
   DB의 CRLF와 호환이 안되는 건가 가정을 해보았다. 아니었다.
   workbench 상으로는 엔터와 스페이스가 구분이 안 되는 듯 했지만,
   select를 통하여 logging해보니 spring <-> db간에는 문제가 없었다.
   
   uri를 확인해보았다. %0A(\n)가 %20(스페이스)로 바뀌는 문제가 발생하였다.
   자바스크립트의 encodingUriComponent였던가.. 이렇게 생긴 놈들에 대해서
   계속 구글링하는데, 뭔가 잘못 짚은 느낌이 들었다.
   jstl의 <c:out>이 문제일 수도 있겠다는 생각이 들어서 검색을 해보았다.
   '<c:out> \n'이런식으로 검색하여 stackOverFlow같은 사이트를 대충 해석하였다.
   <c:out>이 쓰이는 곳마다 계속해서 br태그로 바꾸어주는 방법이 대부분이었다.
   
   계속 여행을 하다, br태그로 바꾸어 주거나 textarea에 출력하라는 내용이 있었다.
   p, input태그에서 값을 넣어주니 라인피드가 씹히는 것이었다.
   textarea에 출력하여 성공하였고, 처음부터 아무런 문제도 없었던 상황이었다...
  
<br/>

> 🚀 Model

```java
@Data
public class Post {
    private Long id;
    private String text;
}
```

> 🚀 Dao / test

```java
//production code
    @Insert("insert into markdown(text) values(#{text})")
    public void save(Post post);

    @Select("select * from markdown")
    public List<Post> getAll();
    
    @Update("update markdown set text = #{text} where id = #{id}")
    public void update(Post post);
    
    @Select("select * from markdown where id = #{id}")
    public Post get(Long id);
```

```java
//test code
    @Test
    public void testGetAll() {
        dao.getAll().forEach((post) -> log.info(post));
    }
    
    @Test
    public void testSave() {
        Post post = new Post();
        post.setText("test......");
        dao.save(post);
    }
    
    @Test
    public void testUpdate() {
        Post post = new Post();
        post.setId(19l);
        post.setText("updated");
        
        //dao.update(post);        
    }
    
    @Test
    public void testGet() {
        dao.get(19l);
    }
```

<br/>

> 🚀 Service (ServiceImpl과 test생략)

```java
    String getTime();
    
    //post가 id가 null이면 insert, 있으면 update로 작동하게 하였다.
    //이번 코드는 전체적으로 샘플을 위하여 대충 만든 것이므로
    //명명법이나 기타 '왜 이렇게 했지'의 생각은 하지 말자
    void save(Post post);

    List<Post> getAll();
    
    public Post get(Long id);
```

> 🚀 Controller

```java
    @GetMapping("/list")
    public void list(Model model) {
        List<Post> texts = service.getAll();

        model.addAttribute("list", texts);
    }

    @GetMapping("/read")
    public void read(Long id, Model model) {
        model.addAttribute("post",service.get(id));
    }

	//수정 또는 작성 폼을 보여주는 부분(이름 대충 지음)
    @GetMapping("/md")
    public void test(Post post) {
    }

	//처리 & 리다이렉트라 get방식이 부적절할 수 있으나
    //샘플이니 대충 만들었다
    @GetMapping("/markdownAfterSave")
    public String test3(Post post) {
        service.save(post);
        return "redirect:/list";
    }
```

> 🚀 view : body만 담았다

1. list.jsp

```jsp
	<div style="text-align: center; border: 1px solid black;">
	
		<c:forEach items="${list}" var="obj">
		
			<div>
				<span id="d" style="color: orange; border: 2px solid red;"><c:out
						value="${obj.id}"></c:out></span>
				<p id=>
					<c:out value="${obj.text}"></c:out>
				</p>
				
				<br />
				
			</div>
			
		</c:forEach>
		
	</div>
	
	<div style="text-align: center;">
		<button id="write">글쓰기</button>
	</div>
	
	<script>
		document.querySelector("#write").addEventListener("click",()=>{location.href = "/md";});
		
		const ps = document.querySelectorAll("#d");
		
		for(var i=0;i<ps.length;i++){
			ps[i].addEventListener("click",(e) => {
				location.href = '/read?id='+encodeURIComponent(e.target.innerText);
			});
		}
	</script>
```

<br/>

2. read.jsp

```jsp
	<div id="view" style="border: 1px solid black;"></div>

	<br />

	<form action="/md">
		<input hidden name="id" readOnly value="${post.id }" />
		<textarea style="width: 600px; height: 300px; resize: none" hidden	name="text" value="">${post.text }</textarea>
		<button type="submit">수정하기</button>
	</form>

	<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
	<script type="text/javascript">
		document.querySelector("#view").innerHTML = marked.parse(document.querySelector("textarea").value);
	</script>
```

<br/>

3. md.jsp

```jsp
	<div style="width: 100%;">
		<div style="width: 50%; float: left;">
			<form action="/markdownAfterSave">
				<input hidden name="id" value="${post.id}" />
				<textarea style="width: 90%; height: 700px; resize: none"
					name="text" id='before'>${post.text}</textarea>
				<br />
				<button type="submit">저장하기</button>
			</form>
		</div>

		<div id='after' style="width: 50%; float: right;"></div>
	</div>
	<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
	<script type="text/javascript">	
		const before = document.querySelector("#before");//텍스트를 입력할 상자
		const after = document.querySelector("#after");//렌더링 된 html을 보여줄 영역
		const save = document.querySelector("#save");//버튼
		
		after.innerHTML = marked.parse(before.value);//초기 렌더링
		
		before.addEventListener("input",() => {after.innerHTML = marked.parse(before.value);});//텍스트 상자가 입력될 때마다 a를 실행한다.
		
	</script>
```

<br/>

> 🚀 결과 화면

1. /list

![](https://images.velog.io/images/sonchanwoo/post/95a1a40c-2c3c-428b-8feb-72f3b9118751/image.png)

2. /read?id=24

![](https://images.velog.io/images/sonchanwoo/post/ce659ff2-5ab0-498b-8d49-071274f34e0f/image.png)

3. /md?id=?&&text=? (수정에 해당)

![](https://images.velog.io/images/sonchanwoo/post/2024c6e0-8328-46a8-90a6-1af0876a17f9/image.png)

- 데이터 그대로 유지도 되고, 실시간 렌더링(우측)도 돼서 넘 좋다.

<br/>

4. /md (작성에 해당)

![](https://images.velog.io/images/sonchanwoo/post/03aa0351-251c-4111-b5aa-28c87b36e92c/image.png)

<br/>

이상!