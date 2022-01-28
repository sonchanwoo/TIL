> 🚀 언제 필요할까?

1. 엔티티

```java
@Data
public class BoardVO {
    private Long bno;
    private String title;
    private String content;
    private String writer;
    private Date regDate;
    private Date updateDate;
}
```

2. 화면

![](https://images.velog.io/images/sonchanwoo/post/8fb932e6-de3a-468f-a04d-ad34a73f9883/3_needs_register.png)

3. 컨트롤러

```java
    @PostMapping("/register")
    public String register(BoardVO vo) {
        service.register(vo);
        return "redirect:/board/list";
    }
```

4. 서비스 / DAO / 쿼리(mapper.xml)

```java
public interface BoardService {
    //구현체에서 dao의 insert()사용
    public void register(BoardVO vo);
}

public interface BoardDAO {
    //xml과 연동
    public int insert(BoardVO vo);
    
}
```

```xml
<insert id="insert">
	insert into tbl_board (title,content,writer)
	values
	(#{title},#{content}, #{writer})
</insert>
```

-> 화면에서 title,content,writer를 입력받아 컨트롤러로 전송한다.

-> VO의 6개의 필드 중 위에서 입력한 세 개 말고는 null이다.

-> regDate, updateDate는 default가 있어 괜찮다.( now() )

-> bno 역시 DB테이블 상 pk, auto_increment로 만들어져 있어 괜찮다.

-> 다만, 등록(register) 후 방금 등록된 게시글의 bno값이 무엇인지 알아야 할 때가 있다.(모달 창 등등)

-> auto_increment는 DB테이블에서 적용되는 것이지, vo의 bno는 null 그대로이다.

-> null이었던 bno의 값을 테이블에 들어간(또는 들어갈) 값으로 넣어주고 싶다.

-> __ 그래서 mybatis의 @SelectKey를 사용한다. __

<br/>

> 🚀 @SelectKey를 사용하는 2가지 방법

1. insert하기 __"전에"__ 다음 bno값을 생성하여 vo에도 주고, insert into table(bno,title,content,writer)한다.

- DAO 메서드 추가

```java
public interface BoardDAO {
    public int insert(BoardVO vo);
    
    //추가
    public int insertSelectKey(Board vo);
    
}
```

- xml 추가

```xml
<insert id="insert">
	insert into tbl_board (title,content,writer)
	values
	(#{title},#{content}, #{writer})
</insert>

<!-- 추가된 부분 -->
<insert id="insertSelectKey">
	<selectKey resultType="long" keyProperty="bno" 	order="BEFORE">
			select MAX(bno) + 1 from tbl_board
	</selectKey>
	insert into tbl_board (bno,title,content,writer)
	values
	(#{bno}, #{title},#{content}, #{writer})
</insert>
```

<br/>

2. insert하고 __"나서"__ 방금 등록된 번호

```xml
<insert id="insertSelectKey">
  
	insert into tbl_board (title,content,writer)
	values
	(#{title},#{content}, #{writer})
		
	<selectKey resultType="long" keyProperty="bno" 	order="AFTER">
		select MAX(bno) from tbl_board
	</selectKey>
  
</insert>
```
- 여기서 MAX(pk) 대신, last_insert_id를 이용할 수도 있지만, 트랜잭션과 관련되어 있어 나중에 다루어 보도록 하겠다.
