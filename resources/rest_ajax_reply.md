> 출처 : 코드로 배우는 스프링 웹 프로젝트(구멍가게 코딩단)

[참고 : rest+ajax 기본](https://github.com/sonchanwoo/TIL/blob/main/resources/rest_basic.md)

---

> 🚀 영속계층

1. VO

```java
@Data
public class ReplyVO {
  private Long rno;
  private Long bno;

  private String reply;
  private String replyer;
  private Date replyDate;
  private Date updateDate;
}
```

2. DAO

```java
public interface ReplyDAO {
    public int insert(ReplyVO vo);

    public ReplyVO read(Long bno);

    public int delete(Long bno);

    public int update(ReplyVO reply);
    
    public List<ReplyVO> getListWithPaging(
    @Param("cri") Criteria cri, @Param("bno") Long bno);
    
    public int getCountByBno(Long bno);
}
```

3. mapper

```xml
<mapper namespace="org.zerock.dao.ReplyDAO">

	<insert id="insert">

		insert into tbl_reply (bno, reply, replyer)
		values (#{bno}, #{reply}, #{replyer})

	</insert>

	<select id="read" resultType="ReplyVO">

		select rno,bno,reply,replyer,replyDate,updateDate
        from tbl_reply where rno = #{rno}

	</select>
  
    <select id="getListWithPaging"
		resultType="ReplyVO">
		select /*+index(tbl_reply idx_reply)*/
        rno, bno, reply, replyer, replyDate, updatedate
		from tbl_reply
		where bno = #{bno}
		limit ${(cri.pageNum-1)*cri.amount}, #{cri.amount}
	</select>
  
    <select id="getCountByBno" resultType="int">
		select count(rno) from tbl_reply where bno = #{bno}
	</select>


	<delete id="delete">

		delete from tbl_reply where rno = #{rno}

	</delete>

	<update id="update">

		update tbl_reply set reply = #{reply},updatedate =
		now()
		where rno = #{rno}

	</update>

</mapper> 
```

<br/>

> 🚀 컨트롤러(서비스 생략)

```java
@RequestMapping("/replies/")
@RestController
@AllArgsConstructor
public class ReplyController {
    private ReplyService service;

    @PostMapping(value = "/new", consumes = "application/json", produces = { MediaType.TEXT_PLAIN_VALUE })
    public ResponseEntity<String> create(@RequestBody ReplyVO vo) {
        int insertCount = service.register(vo);

        return insertCount == 1 ? new ResponseEntity<>("success", HttpStatus.OK)
                : new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @GetMapping(value = "/pages/{bno}/{page}", produces = { MediaType.APPLICATION_XML_VALUE,
            MediaType.APPLICATION_JSON_UTF8_VALUE })
    public ResponseEntity<List<ReplyVO>> getList(@PathVariable("page") int page, @PathVariable("bno") Long bno) {

        Criteria cri = new Criteria(page, 10);

        return new ResponseEntity<>(service.getList(cri, bno), HttpStatus.OK);
    }

    @GetMapping(value = "/{rno}", produces = { MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_UTF8_VALUE })
    public ResponseEntity<ReplyVO> get(@PathVariable("rno") Long rno) {

        return new ResponseEntity<>(service.get(rno), HttpStatus.OK);
    }

    @RequestMapping(method = { RequestMethod.PUT,
            RequestMethod.PATCH }, value = "/{rno}", consumes = "application/json", produces = {
                    MediaType.TEXT_PLAIN_VALUE })
    public ResponseEntity<String> modify(@RequestBody ReplyVO vo, @PathVariable("rno") Long rno) {

        vo.setRno(rno);

        return service.modify(vo) == 1 ? new ResponseEntity<>("success", HttpStatus.OK)
                : new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);

    }

    @DeleteMapping(value = "/{rno}", produces = { MediaType.TEXT_PLAIN_VALUE })
    public ResponseEntity<String> remove(@PathVariable("rno") Long rno) {

        return service.remove(rno) == 1 ? new ResponseEntity<>("success", HttpStatus.OK)
                : new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);

    }
}
```

<br/>

> 🚀 (rest+ajax)댓글 서비스 구현

1. reply.js

```javascript
const replyService = (function(){
  
    //add
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
    
    //getList
	function getList(param, callback, error) {

	    var bno = param.bno;
	    var page = param.page || 1;//없으면 무적권 1이라는뜻

	    $.getJSON("/replies/pages/" + bno + "/" + page + ".json",
	        function(data) {
	          if (callback) {
	            callback(data);
	          }
	        }).fail(function(xhr, status, err) {
	      if (error) {
	        error();
	      }
	    });
	  }

    //remove
	function remove(rno, callback, error) {
		$.ajax({
			type : 'delete',
			url : '/replies/' + rno,
			success : function(deleteResult, status, xhr) {
				if (callback) {
					callback(deleteResult);
				}
			},
			error : function(xhr, status, er) {
				if (error) {
					error(er);
				}
			}
		});
	}

    //update
	function update(reply, callback, error) {
		$.ajax({
			type : 'put',
			url : '/replies/' + reply.rno,
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
			}
		});
	}

    //get
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

    //date-format용
	function displayTime(timeValue) {

		var today = new Date();

		var gap = today.getTime() - timeValue;

		var dateObj = new Date(timeValue);
		var str = "";

		if (gap < (1000 * 60 * 60 * 24)) {

			var hh = dateObj.getHours();
			var mi = dateObj.getMinutes();
			var ss = dateObj.getSeconds();

			return [ (hh > 9 ? '' : '0') + hh, ':', (mi > 9 ? '' : '0') + mi,
					':', (ss > 9 ? '' : '0') + ss ].join('');

		} else {
			var yy = dateObj.getFullYear();
			var mm = dateObj.getMonth() + 1; // getMonth() is zero-based
			var dd = dateObj.getDate();

			return [ yy, '/', (mm > 9 ? '' : '0') + mm, '/',
					(dd > 9 ? '' : '0') + dd ].join('');
		}
	}

	return {
		add : add,
		get : get,
		getList : getList,
		remove : remove,
		update : update,
		displayTime : displayTime
	};

})();
```

2. get.jsp

- 댓글 목록 화면 추가

```html
<div class='row'>
	<div class="col-lg-12">
		<div class="panel panel-default">
        
            <!-- 등록버튼을 포함한 헤더 -->
			<div class="panel-heading">
				<i class="fa fa-comments fa-fw"></i> Reply
				<button id='addReplyBtn' class='btn btn-primary btn-xs pull-right'>New
					Reply</button>
			</div>
            
			<!-- 댓글 내용 바디 -->
			<div class="panel-body">
				<ul class="chat">

				</ul>
			</div>
            
			<!-- 풋터 -->
			<div class="panel-footer"></div>

		</div>
	</div>
</div>
```

- 댓글 등록 용 모달 추가

```html
<!-- Modal -->
<div class="modal fade" id="myModal" tabindex="-1" role="dialog"
	aria-labelledby="myModalLabel" aria-hidden="true">
	<div class="modal-dialog">
		<div class="modal-content">
			<div class="modal-header">
				<button type="button" class="close" data-dismiss="modal"
					aria-hidden="true">&times;</button>
				<h4 class="modal-title" id="myModalLabel">REPLY MODAL</h4>
			</div>
          
            <!-- 정보들이 있는 바디 -->
			<div class="modal-body">
				<div class="form-group">
					<label>Reply</label>
                    <input class="form-control" name='reply' value='New Reply!!!!' />
				</div>
				<div class="form-group">
					<label>Replyer</label>
                  <input class="form-control" name='replyer' value='replyer' />
				</div>
				<div class="form-group">
					<label>Reply Date</label>
                    <input class="form-control" name='replyDate' value='2018-01-01 13:13' />
				</div>

			</div>
          
            <!-- 버튼들이 있는 풋터 -->
			<div class="modal-footer">
				<button id='modalModBtn' type="button" class="btn btn-warning">Modify</button>
				<button id='modalRemoveBtn' type="button" class="btn btn-danger">Remove</button>
				<button id='modalRegisterBtn' type="button" class="btn btn-primary">Register</button>
				<button id='modalCloseBtn' type="button" class="btn btn-default" data-dismiss="modal">Close</button>
			</div>
		</div>
		<!-- /.modal-content -->
	</div>
	<!-- /.modal-dialog -->
</div>
<!-- /.modal -->
```

- 이벤트처리

```javascript
//replyService 포함
<script type="text/javascript" src="/resources/js/reply.js"></script>

<script type="text/javascript">
	$(document).ready(function() {
		var bnoValue = '<c:out value="${vo.bno}"/>';
		var replyUL = $(".chat");
		
  		showList(1);
  		
  		// /pages/{bno}/{page}
		function showList(page) {
			replyService.getList(
				{bno : bnoValue,
				 page : page || 1},
              
				 //콜백함수
                 function(list) {
				 	var str = "";
                   
					if (list == null|| list.length == 0) {
						replyUL.html("");
						return;
					}
							
                    for (var i = 0, len = list.length || 0; i < len; i++) {
						str += "<li class='left clearfix' data-rno='"+list[i].rno+"'>";
						str += "  <div><div class='header'><strong class='primary-font'>"+ list[i].replyer + "</strong>";
						str += "    <small class='pull-right text-muted'>"+ replyService.displayTime(list[i].replyDate)	+ "</small></div>";
						str += "    <p>"+ list[i].reply + "</p></div></li>";
					}
					
                    replyUL.html(str);
				 });//end function
		 }//end showList 
						
		var modal = $(".modal");
		var modalInputReply = modal.find("input[name='reply']");
		var modalInputReplyer = modal.find("input[name='replyer']");
		var modalInputReplyDate = modal.find("input[name='replyDate']");
					    
		var modalModBtn = $("#modalModBtn");
		var modalRemoveBtn = $("#modalRemoveBtn");
		var modalRegisterBtn = $("#modalRegisterBtn");
		
  		//댓글 작성 버튼 클릭시 모달 버튼 보여주기
		$("#addReplyBtn").on("click", function(e){
            //버튼이 4종류 다 있기 때문에 필요한 것만 띄운다
			modal.find("input").val("");
			modalInputReplyDate.closest("div").hide();
			modal.find("button[id !='modalCloseBtn']").hide();
					        
			modalRegisterBtn.show();
					        
			$(".modal").modal("show");
					        
		});
		
  		//등록버튼 클릭 시 /reply/new 실행(reply,replyer,bno줘야함)
		modalRegisterBtn.on("click",function(e){
			var reply = {
					      reply: modalInputReply.val(),
              			  replyer:modalInputReplyer.val(),
					      bno:bnoValue
					     };
					        
            replyService.add(reply, function(result){
                //콜백함수 : 성공결과 + 내용 비우기 + 모달 닫기 + 목록보여주기
				alert(result);					          	
              	modal.find("input").val("");
                modal.modal("hide");
					          
				showList(1);					          
			});					        
		});//end-register
  
  		//댓글 조회 클릭 이벤트 처리 
		$(".chat").on("click", "li", function(e){
          	//li태그 작성시 data-rno로 ${rno}를 넣어준데에 이유가 있다.
			var rno = $(this).data("rno");
			
          	//get : rno 를 주면 vo가 반환된다.
            replyService.get(rno, function(reply){
            	modalInputReply.val(reply.reply);
                modalInputReplyer.val(reply.replyer);
                modalInputReplyDate.val(replyService.displayTime( reply.replyDate)).attr("readonly","readonly");
			    modal.data("rno", reply.rno);
					        
				modal.find("button[id !='modalCloseBtn']").hide();
				modalModBtn.show();
				modalRemoveBtn.show();
					        
				$(".modal").modal("show");
			});
		});
					  
					  
		//수정
		modalModBtn.on("click", function(e){
			var reply = {rno:modal.data("rno"), reply: modalInputReply.val()};
          
            replyService.update(reply, function(result){
            	alert(result);
				modal.modal("hide");
				showList(1);
					          
			});					        
		});
		
  		//삭제
  		modalRemoveBtn.on("click", function (e){
        	var rno = modal.data("rno");
          
          	replyService.remove(rno, function(result){
            	alert(result);
				modal.modal("hide");
				showList(1);					    	      
			});					    	  
		}); 
					
	});
</script>
```