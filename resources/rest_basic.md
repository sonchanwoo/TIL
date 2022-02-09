> 출처 : 코드로 배우는 스프링 웹 프로젝트(구멍가게 코딩단)

---

> 🚀 @RestController

1. 기본 반환

```java
@RestController
@RequestMapping("/sample")
@Log4j
public class SampleController {

	@GetMapping(value = "/getText", produces = "text/plain; charset=UTF-8")
	public String getText() {

		log.info("MIME TYPE: " + MediaType.TEXT_PLAIN_VALUE);

		return "안녕하세요";

	}
}
```

- /sample/getText 경로

- produces : 메서드가 생성하는 MIME타입

- return 값으로 순수 데이터만을 준다.

- 개발자 도구로 확인!

<br/>

2. 객체 반환

```java
    @GetMapping(value = "/getSample", 
			produces = { MediaType.APPLICATION_JSON_UTF8_VALUE,
			MediaType.APPLICATION_XML_VALUE })
	public SampleVO getSample() {

		return new SampleVO(112, "스타", "로드");

	}
```

- 생성하는 타입을 여러개 줄 수 있다.(생략도 가능)

- /sample/getSample로 호출 시 xml형식, .json을 붙이면 json형식으로 전달된다.

- 컬렉션도 가능하다.(리스트는 배열, 맵은 하나의 객체로 간주된다.)

<br/>

2. 바인딩과 ResponseEntity

```java
    @GetMapping("/product/{cat}/{pid}")
	public String[] getPath(@PathVariable("cat") String cat, @PathVariable("pid") Integer pid) {

		return new String[] { "category: " + cat, "productid: " + pid };
	}

	@PostMapping("/ticket")
	public Ticket convert(@RequestBody Ticket ticket) {

		log.info("convert.......ticket" + ticket);

		return ticket;

	}
```

- path를 이용하고 싶으면, @PathVariabble

- 쿼리스트링을 객체로 받고 싶으면 @ResponseBBody

- body-data뿐 아니라 상태메세지 등도 같이 주고 싶다면 ResponseEntity를 아래와 같이 이용한다.

```java
        ResponseEntity<SampleVO> result = null;

		if (height < 150) {
			result = ResponseEntity.status(HttpStatus.BAD_GATEWAY).body(vo);
		} else {
			result = ResponseEntity.status(HttpStatus.OK).body(vo);
		}

		return result;
        
        //또는
        
        return new ResponseEntity<>(service.get(rno), HttpStatus.OK);
```

<br/>

> 🚀 ajax

- 기존의 @Controller 방식은 주소를 직접 이동하여 처리하였다면, @RestController방식으로는 javascript로 주소를 호출하여 값만 받고(주고) 이 값으로 무언가를 처리하는 방식으로 간다.

1. 기본 흐름(데이터를 넘겨줄 때 경우)

- reply.js

```javascript
const replyService = (function(){
  
	function add(reply, callback, error) {
		$.ajax({
			type : 'post',
			url : '/replies/new',
			data : JSON.stringify(reply),
			contentType : "application/json; charset=utf-8",
			success : function(result, status, xhr) {
				if (callback) {
					callback(result);
				}
			},
			error : function(xhr, status, er) {
				if (error) {
					error(er);
				}
			},
		})
	}
    return {
		add : add,
	};
})(); 
```

-> replyService.add()하면 add함수가 실행된다.

-> 첫번째 매개변수: 데이터, 두번째 매개변수: 성공시 실행함수, 세번째: 실패시 실행함수 (성공 및 실패 콜백함수는 필요시에만 사용할 수도 있다.)

-> $.ajax()내부에 submit시 필요한 자원들을 적용한다.

-> JSON.stringify()는 js객체를 JSON으로 변환하는 것이다.

-> 성공시에 callback함수를 부를 수 있다.

-> result는 ResponseEntity로 준 "success"를 받아 온다.

<br/>

- get.jsp

```javascript
    <script type="text/javascript" src="/resources/js/reply.js"></script>

	<script type="text/javascript">
	$(document).ready(function(){
		
		const bnoValue = '${vo.bno}';
      
		replyService.add(
		  {reply:"jsonTest",replyer:"tester",bno:bnoValue},
		  function(result){
			  alert(result);
		  }
		);
      
    });
	</script>
```

-> 위에서 만든 reply.js를 포함한다.

-> replyService.add()형식에 맞게, data와 성공시콜백함수를 넘겨준다.

-> 성공시에 콜백함수가 실행된다.

<br/>

2. 기본 흐름(데이터를 받을 때)

```javascript
    function get(rno, callback, error) {

		$.get("/replies/" + rno + ".json", function(result) {

			if (callback) {
				callback(result);
			}

		}).fail(function(xhr, status, err) {
			if (error) {
				error();
			}
		});
	}
```

- $.ajax에서 데이터를 넘겨줄 필요가 없고, get방식일 때 필요없는 부분을 생략하고 만들어진 축약형이다.

- $.get(uri,callback(data));로 사용하면 되며, 에러콜백도 특이하다.

```javascript
    replyService.get(4,function(data){
		console.log(data);
	});	
```

- 사용은 이런식으로 하면 된다.