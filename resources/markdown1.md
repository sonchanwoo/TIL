# markdown 렌더링 예제!

- 프로젝트 참고 대상인 'velog'의 글 작성시 markdown으로 렌더링 하는 부분을 공부해보기로 하였다.

- 구글링을 하다 Octokit이라는 것이 눈에 띄어 관련된 정보들을 찾다가

- await, async 같은 것들이 나오길래 이렇게까지 하는 게 맞나 싶었다.

- 검색어를 markdown api -> markdown renderer로 변경하였다.

- 그랬더니 markedJs라는 번들이 있었다.(..?)

- 소스 코드는 아래와 같다.

```java
	<textarea id = 'before' ></textarea><!-- 텍스트를 입력할 상자 -->
	
	<div id = 'after' style=""></div><!-- 렌더링 된 html을 보여줄 영역 -->
	
	<!-- markdown형식의 plain-text를 html로 rendering 해주기 위한 라이브러리 -->
	<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
	
	<script type="text/javascript">
	
		const before = document.querySelector("#before");//텍스트를 입력할 상자
		const after = document.querySelector("#after");//렌더링 된 html을 보여줄 영역
		
		function a(){
			after.innerHTML = marked.parse(before.value);//텍스트 상자의 내용을 parse하여 html을 구성한다.
		}
		
		before.addEventListener("input",a);//텍스트 상자가 입력될 때마다 a를 실행한다.
		
		/*
		marked.setOptions({
			gfm:true,
		});*/
		
	</script>
```

<br/>

- 화면은 아래와 같다.

![](https://images.velog.io/images/sonchanwoo/post/3a5dde9a-8971-47f8-9d30-bc50442e0fcd/image.png)

<br/>

   기본적인 markdown 문법은 준수하지만, 인용구가 그저 들여쓰기만 되는 것, 코드블럭 언어형식에 따른 색깔 변화를 볼 수 없다는 점 등의 커스텀과 관련된 문제가 있었다.

   기본적으로 html태그로써 렌더링 되기 때문에 css를 건들면 될 것 같아 보인다.

   깃허브 README의 형식과 다른 점(대표적으로 띄어쓰기 문제 등)을 맞추고자 setOption에 대하여 공부하여 볼 생각이다.

<br/>