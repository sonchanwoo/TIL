> 출처 : 코드로 배우는 스프링 웹 프로젝트(구멍가게 코딩단)

---

> 🚀 댓글의 페이징

1. 댓글 목록과 댓글 수를 담는 ReplyPageDTO 작성

```java
@Data
@AllArgsConstructor
@Getter
@ToString
public class ReplyPageDTO {
  private List<ReplyVO> list;

  private int replyCnt;

}
```

2. 서비스의 변화

```java
	@Override
    //public List<ReplyVO> getList(Criteria cri, Long bno) {
    public ReplyPageDTO getList(Criteria cri, Long bno) {

      //return dao.getListWithPaging(cri, bno);
      return new ReplyPageDTO(dao.getListWithPaging(cri, bno),dao.getCountByBno(bno));

    }
```

3. 컨트롤러의 변화

```java
	@GetMapping(value = "/pages/{bno}/{page}", produces = { MediaType.APPLICATION_XML_VALUE,
            MediaType.APPLICATION_JSON_UTF8_VALUE })
    //public ResponseEntity<List<ReplyVO>> getList(@PathVariable("page") int page, @PathVariable("bno") Long bno) {
    public ResponseEntity<ReplyPageDTO> getList(@PathVariable("page") int page, @PathVariable("bno") Long bno) {

        Criteria cri = new Criteria(page, 10);

        return new ResponseEntity<>(service.getList(cri, bno), HttpStatus.OK);
    }
```

4. reply.js의 변화

```javascript
	//getList
	function getList(param, callback, error) {

	    var bno = param.bno;
	    var page = param.page || 1;

	    $.getJSON("/replies/pages/" + bno + "/" + page + ".json",
	        function(data) {
	          if (callback) {
	            //callback(data);
                callback(data.list,data.replyCnt);//이부분이 변화
	          }
	        }).fail(function(xhr, status, err) {
	      if (error) {
	        error();
	      }
	    });
	  }
```

5. get.jsp의 showList() 변화

```javascript
    //function(list) { 파라미터 변경
	function(list,replyCnt) {
		var str = "";
                   
		if (list == null|| list.length == 0) {
			return;
		}
							
        for (var i = 0, len = list.length || 0; i < len; i++) {
			str += "<li class='left clearfix' data-rno='"+list[i].rno+"'>";
			str += "  <div><div class='header'><strong class='primary-font'>"+ list[i].replyer + "</strong>";
			str += "    <small class='pull-right text-muted'>"+ replyService.displayTime(list[i].replyDate)	+ "</small></div>";
			str += "    <p>"+ list[i].reply + "</p></div></li>";
		}
					
        replyUL.html(str);
      
        //댓글용 pageMaker 보여주기
        showReplyPage(replyCnt);
      
	});//end function
```

6. get.jsp에 pageMaker추가

```javascript
	var pageNum = 1;
	var replyPageFooter = $(".panel-footer");
					      
	function showReplyPage(replyCnt){
					        
		var endNum = Math.ceil(pageNum / 10.0) * 10;  
		var startNum = endNum - 9; 
					        
		var prev = startNum != 1;
		var next = false;
					        
		if(endNum * 10 >= replyCnt){
			endNum = Math.ceil(replyCnt/10.0);
		}
					        
		if(endNum * 10 < replyCnt){
			next = true;
		}
					        
		var str = "<ul class='pagination pull-right'>";
					        
		if(prev){
			str+= "<li class='page-item'><a class='page-link' href='"+(startNum -1)+"'>Previous</a></li>";
		}
					        
		for(var i = startNum ; i <= endNum; i++){
					          
			var active = pageNum == i? "active":"";
					          
			str+= "<li class='page-item "+active+" '><a class='page-link' href='"+i+"'>"+i+"</a></li>";
		}
					        
		if(next){
			str+= "<li class='page-item'><a class='page-link' href='"+(endNum + 1)+"'>Next</a></li>";
		}
					        
		str += "</ul></div>";
					        
		replyPageFooter.html(str);
	}
					       
	replyPageFooter.on("click","li a", function(e){
		e.preventDefault();
					         
		var targetPageNum = $(this).attr("href");
					         
		pageNum = targetPageNum;
					         
		showList(pageNum);
	});
```