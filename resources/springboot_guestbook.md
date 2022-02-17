# spring boot + thymeleaf + jpa로 간단한 방명록 예제 만들기

> 출처 : 코드로 배우는 스프링 부트 웹 프로젝트(구멍가게 코딩단)

---

> 🚀 entity

```java
@MappedSuperclass//테이블로 생성되지 않게 막는다.
@EntityListeners(value = {AuditingEntityListener.class})
@Getter
abstract class BaseEntity {
	//이벤트를 감지하여 처리를 한다(두번째 어노테이션과 
    //메인메서드가 있는 application의 @EnableJpaAuditing의 합작)

	@CreatedDate//자동으로 현재 시간 생성
	@Column(name = "regdate", updatable = false)
	private LocalDateTime regDate;

	@LastModifiedDate//변화를 감지하여 수정 시간 설정
	@Column(name = "moddate")
	private LocalDateTime modDate;

}
```

```java
@Entity
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class Guestbook extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long gno;

    @Column(length = 100, nullable = false)
    private String title;

    @Column(length = 1500, nullable = false)
    private String content;

    @Column(length = 50, nullable = false)
    private String writer;

    public void changeTitle(String title){
        this.title = title;
    }

    public void changeContent(String content){
        this.content = content;
    }
}
```

- 실제로 테이블에 생성될 Entity이다.

- 위 BaseEntity를 상속받는다.

<br/>

> 🚀 Querydsl

- 쿼리문을 코드로 작성할 수 있게 해주는 오픈소스 프레임워크이다.

- 설정(2.6버전이 어쨌니 저쨌니 때문에 고생했었다)

```gradle
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
	...

    //queryDsl
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

...

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {

    ...   

    // QueryDSL
    implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"

    annotationProcessor(
            "javax.persistence:javax.persistence-api",
            "javax.annotation:javax.annotation-api",
            "com.querydsl:querydsl-apt:${queryDslVersion}:jpa")

	...
}

...

def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```

- 코끼리 갱신 후 compile옵션 옆에 시작버튼을 누르면 엔티티클래스를 Q도메인으로 변환하여 build라는 폴더에 엔티티 객체와 같은 구조로 Q~가 생긴다.

- qEntity = Qdomain.entity, qEntity.id 와 같은 식으로 직접 접근이 가능하다.

<br/>

- 간단 사용법

1. 목표 쿼리

```
Hibernate: 
    select
        guestbook0_.gno as gno1_0_,
        guestbook0_.moddate as moddate2_0_,
        guestbook0_.regdate as regdate3_0_,
        guestbook0_.content as content4_0_,
        guestbook0_.title as title5_0_,
        guestbook0_.writer as writer6_0_ 
    from
        guestbook guestbook0_ 
    where
        guestbook0_.title like ? escape '!' 
    order by
        guestbook0_.gno desc limit ?
```

2. 코드

```java
        Pageable pageable = PageRequest.of(0, 10, Sort.by("gno").descending());

        QGuestbook qGuestbook = QGuestbook.guestbook;

        String keyword = "1";

        BooleanBuilder builder = new BooleanBuilder();
        //where절

        BooleanExpression expression = qGuestbook.title.contains(keyword);
        //title like '%1%'

        builder.and(expression);
        //합체(mybatis의 젤 앞 문장은 예약어가 필요하지 않는 처리를 간단히 할 수 있어서 좋다)
        //괄호처리도 바인딩 순서만 잘 해주면 된다는 점이 편하다.
        //BooleanBuilder뿐 아니라 ~Expression에도 and,or가 있다.
        //builder.and(qGuestbook.gno.gt(0L)); 이런식으로 gno>0조건도 줄 수 있다.

        Page<Guestbook> result = guestbookRepository.findAll(builder, pageable);
        //findAll의 이런 오버로딩도 있음

        result.stream().forEach(guestbook -> {
            System.out.println(guestbook);
        });
```

<br/>

> 🚀 repository (dao / mapper)

```java
public interface GuestbookRepository extends JpaRepository<Guestbook, Long>, 
	QuerydslPredicateExecutor<Guestbook> {
}
```

<br/>

> 🚀 service

```java
public interface GuestbookService {

	Long register(GuestbookDTO dto);

	PageResultDTO<GuestbookDTO, Guestbook> getList(PageRequestDTO requestDTO);

	GuestbookDTO read(Long gno);

	void modify(GuestbookDTO dto);

	void remove(Long gno);

	default Guestbook dtoToEntity(GuestbookDTO dto) {
		Guestbook entity = Guestbook.builder()
			.gno(dto.getGno())
			.title(dto.getTitle())
			.content(dto.getContent())
			.writer(dto.getWriter())
			.build();
		return entity;
	}

	default GuestbookDTO entityToDto(Guestbook entity) {

		GuestbookDTO dto = GuestbookDTO.builder()
			.gno(entity.getGno())
			.title(entity.getTitle())
			.content(entity.getContent())
			.writer(entity.getWriter())
			.regDate(entity.getRegDate())
			.modDate(entity.getModDate())
			.build();

		return dto;
	}
}
```

```java
@Service
@Log4j2
@RequiredArgsConstructor
public class GuestbookServiceImpl implements GuestbookService {

	private final GuestbookRepository repository;

	@Override
	public Long register(GuestbookDTO dto) {

		log.info("DTO------------------------");
		log.info(dto);

		Guestbook entity = dtoToEntity(dto);

		log.info(entity);

		repository.save(entity);

		return entity.getGno();
	}

	@Override
	public PageResultDTO<GuestbookDTO, Guestbook> getList(PageRequestDTO requestDTO) {

		Pageable pageable = requestDTO.getPageable(Sort.by("gno").descending());

		BooleanBuilder booleanBuilder = getSearch(requestDTO); //검색 조건 처리

		Page<Guestbook> result = repository.findAll(booleanBuilder, pageable); //Querydsl 사용

		Function<Guestbook, GuestbookDTO> fn = (entity -> entityToDto(entity));

		return new PageResultDTO<>(result, fn);
	}

	@Override
	public GuestbookDTO read(Long gno) {

		Optional<Guestbook> result = repository.findById(gno);

		return result.isPresent() ? entityToDto(result.get()) : null;
	}

	@Override
	public void remove(Long gno) {

		repository.deleteById(gno);

	}

	@Override
	public void modify(GuestbookDTO dto) {

		//업데이트 하는 항목은 '제목', '내용'

		Optional<Guestbook> result = repository.findById(dto.getGno());

		if (result.isPresent()) {

			Guestbook entity = result.get();

			entity.changeTitle(dto.getTitle());
			entity.changeContent(dto.getContent());

			repository.save(entity);

		}
	}

	private BooleanBuilder getSearch(PageRequestDTO requestDTO) {

		String type = requestDTO.getType();

		BooleanBuilder booleanBuilder = new BooleanBuilder();

		QGuestbook qGuestbook = QGuestbook.guestbook;

		String keyword = requestDTO.getKeyword();

		BooleanExpression expression = qGuestbook.gno.gt(0L); // gno > 0 조건만 생성

		booleanBuilder.and(expression);

		if (type == null || type.trim().length() == 0) { //검색 조건이 없는 경우
			return booleanBuilder;
		}

		//검색 조건을 작성하기
		BooleanBuilder conditionBuilder = new BooleanBuilder();

		if (type.contains("t")) {
			conditionBuilder.or(qGuestbook.title.contains(keyword));
		}
		if (type.contains("c")) {
			conditionBuilder.or(qGuestbook.content.contains(keyword));
		}
		if (type.contains("w")) {
			conditionBuilder.or(qGuestbook.writer.contains(keyword));
		}

		//모든 조건 통합
		booleanBuilder.and(conditionBuilder);

		return booleanBuilder;
	}

}
```

<br/>

> 🚀 controller

```java
@Controller
@RequestMapping("/guestbook")
@Log4j2
@RequiredArgsConstructor
public class GuestbookController {

	private final GuestbookService service;

	@GetMapping("/")
	public String index() {

		return "redirect:/guestbook/list";
	}

	@GetMapping("/list")
	public void list(PageRequestDTO pageRequestDTO, Model model) {

		log.info("list............." + pageRequestDTO);

		model.addAttribute("result", service.getList(pageRequestDTO));

	}

	@GetMapping("/register")
	public void register() {
		log.info("regiser get...");
	}

	@PostMapping("/register")
	public String registerPost(GuestbookDTO dto, RedirectAttributes redirectAttributes) {

		log.info("dto..." + dto);

		//새로 추가된 엔티티의 번호
		Long gno = service.register(dto);

		redirectAttributes.addFlashAttribute("msg", gno);

		return "redirect:/guestbook/list";
	}

	@GetMapping({"/read", "/modify"})
	public void read(long gno, @ModelAttribute("requestDTO") PageRequestDTO requestDTO, Model model) {

		log.info("gno: " + gno);

		GuestbookDTO dto = service.read(gno);

		model.addAttribute("dto", dto);

	}

	@PostMapping("/remove")
	public String remove(long gno, RedirectAttributes redirectAttributes) {

		log.info("gno: " + gno);

		service.remove(gno);

		redirectAttributes.addFlashAttribute("msg", gno);

		return "redirect:/guestbook/list";

	}

	@PostMapping("/modify")
	public String modify(GuestbookDTO dto,
		@ModelAttribute("requestDTO") PageRequestDTO requestDTO,
		RedirectAttributes redirectAttributes) {

		log.info("post modify.........................................");
		log.info("dto: " + dto);

		service.modify(dto);

		redirectAttributes.addAttribute("page", requestDTO.getPage());
		redirectAttributes.addAttribute("type", requestDTO.getType());
		redirectAttributes.addAttribute("keyword", requestDTO.getKeyword());

		redirectAttributes.addAttribute("gno", dto.getGno());

		return "redirect:/guestbook/read";

	}

}
```

<br/>

> 🚀 화면(일부)(thymeleaf와 제이쿼리 사용법 익힐 것)

1. list.html

```html
		<form action="/guestbook/list" method="get" id="searchForm">
            <div class="input-group">
                <input type="hidden" name="page" value = "1">
                <div class="input-group-prepend">
                    <select class="custom-select" name="type">
                        <option th:selected="${pageRequestDTO.type == null}">-------</option>
                        <option value="t" th:selected="${pageRequestDTO.type =='t'}" >제목</option>
                        <option value="c" th:selected="${pageRequestDTO.type =='c'}"  >내용</option>
                        <option value="w"  th:selected="${pageRequestDTO.type =='w'}" >작성자</option>
                        <option value="tc"  th:selected="${pageRequestDTO.type =='tc'}" >제목 + 내용</option>
                        <option value="tcw"  th:selected="${pageRequestDTO.type =='tcw'}" >제목 + 내용 + 작성자</option>
                    </select>
                </div>
                <input class="form-control" name="keyword" th:value="${pageRequestDTO.keyword}">
                <div class="input-group-append" id="button-addon4">
                    <button class="btn btn-outline-secondary btn-search" type="button">Search</button>
                    <button class="btn btn-outline-secondary btn-clear" type="button">Clear</button>
                </div>
            </div>
        </form>



        <table class="table table-striped">
            <thead>
            <tr>
                <th scope="col">#</th>
                <th scope="col">Title</th>
                <th scope="col">Writer</th>
                <th scope="col">Regdate</th>
            </tr>
            </thead>
            <tbody>

            <tr th:each="dto : ${result.dtoList}" >
                <th scope="row">
                    <a th:href="@{/guestbook/read(gno = ${dto.gno},
                    page= ${result.page},
                    type=${pageRequestDTO.type} ,
                    keyword = ${pageRequestDTO.keyword})}">
                        [[${dto.gno}]]
                    </a>
                </th>
                <td>[[${dto.title}]]</td>
                <td>[[${dto.writer}]]</td>
                <td>[[${#temporals.format(dto.regDate, 'yyyy/MM/dd')}]]</td>
            </tr>



            </tbody>
        </table>

        <ul class="pagination h-100 justify-content-center align-items-center">

            <li class="page-item " th:if="${result.prev}">
                <a class="page-link" th:href="@{/guestbook/list(page= ${result.start -1},
                    type=${pageRequestDTO.type} ,
                    keyword = ${pageRequestDTO.keyword} ) }" tabindex="-1">Previous</a>
            </li>

            <li th:class=" 'page-item ' + ${result.page == page?'active':''} " th:each="page: ${result.pageList}">
                <a class="page-link" th:href="@{/guestbook/list(page = ${page} ,
                   type=${pageRequestDTO.type} ,
                   keyword = ${pageRequestDTO.keyword}  )}">
                    [[${page}]]
                </a>
            </li>

            <li class="page-item" th:if="${result.next}">
                <a class="page-link" th:href="@{/guestbook/list(page= ${result.end + 1} ,
                    type=${pageRequestDTO.type} ,
                    keyword = ${pageRequestDTO.keyword} )}">Next</a>
            </li>

        </ul>


        <div class="modal" tabindex="-1" role="dialog">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Modal title</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <p>Modal body text goes here.</p>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                        <button type="button" class="btn btn-primary">Save changes</button>
                    </div>
                </div>
            </div>
        </div>

        <script th:inline="javascript">

            var msg = [[${msg}]];

            console.log(msg);

            if(msg){
                $(".modal").modal();
            }
            var searchForm = $("#searchForm");

            $('.btn-search').click(function(e){

                searchForm.submit();

            });

            $('.btn-clear').click(function(e){

                searchForm.empty().submit();

            });


        </script>
```

<br/>

2. modify.html

```html
        <form action="/guestbook/modify" method="post">

            <!--페이지 번호  -->
            <input type="hidden" name="page" th:value="${requestDTO.page}">
            <input type="hidden" name="type" th:value="${requestDTO.type}" >
            <input type="hidden" name="keyword" th:value="${requestDTO.keyword}" >


            <div class="form-group">
            <label >Gno</label>
            <input type="text" class="form-control" name="gno" th:value="${dto.gno}" readonly >
        </div>

        <div class="form-group">
            <label >Title</label>>
            <input type="text" class="form-control" name="title" th:value="${dto.title}" >
        </div>
        <div class="form-group">
            <label >Content</label>
            <textarea class="form-control" rows="5" name="content">[[${dto.content}]]</textarea>
        </div>
        <div class="form-group">
            <label >Writer</label>
            <input type="text" class="form-control" name="writer" th:value="${dto.writer}" readonly>
        </div>
        <div class="form-group">
            <label >RegDate</label>
            <input type="text" class="form-control" th:value="${#temporals.format(dto.regDate, 'yyyy/MM/dd HH:mm:ss')}" readonly>
        </div>
        <div class="form-group">
            <label >ModDate</label>
            <input type="text" class="form-control" th:value="${#temporals.format(dto.modDate, 'yyyy/MM/dd HH:mm:ss')}" readonly>
        </div>

        </form>

        <button type="button" class="btn btn-primary modifyBtn">Modify</button>

        <button type="button" class="btn btn-info listBtn">List</button>

        <button type="button" class="btn btn-danger removeBtn">Remove</button>

        <script th:inline="javascript">

            var actionForm = $("form"); //form 태그 객체

            $(".removeBtn").click(function(){

                actionForm
                    .attr("action", "/guestbook/remove")
                    .attr("method","post");

                actionForm.submit();

            });

            $(".modifyBtn").click(function() {

                if(!confirm("수정하시겠습니까?")){
                    return ;
                }

                actionForm
                    .attr("action", "/guestbook/modify")
                    .attr("method","post")
                    .submit();
            });

            $(".listBtn").click(function() {

                //var pageInfo = $("input[name='page']");
                var page = $("input[name='page']");
                var type = $("input[name='type']");
                var keyword = $("input[name='keyword']");

                actionForm.empty(); //form 태그의 모든 내용을 지우고

                actionForm.append(page);
                actionForm.append(type);
                actionForm.append(keyword);


                actionForm
                    .attr("action", "/guestbook/list")
                    .attr("method","get");

               actionForm.submit();

            })

        </script>
```

<br/>

3. read.html

```html
        <div class="form-group">
            <label >Gno</label>
            <input type="text" class="form-control" name="gno" th:value="${dto.gno}" readonly >
        </div>

        <div class="form-group">
            <label >Title</label>>
            <input type="text" class="form-control" name="title" th:value="${dto.title}" readonly >
        </div>
        <div class="form-group">
            <label >Content</label>
            <textarea class="form-control" rows="5" name="content" readonly>[[${dto.content}]]</textarea>
        </div>
        <div class="form-group">
            <label >Writer</label>
            <input type="text" class="form-control" name="writer" th:value="${dto.writer}" readonly>
        </div>
        <div class="form-group">
            <label >RegDate</label>
            <input type="text" class="form-control" name="regDate" th:value="${#temporals.format(dto.regDate, 'yyyy/MM/dd HH:mm:ss')}" readonly>
        </div>
        <div class="form-group">
            <label >ModDate</label>
            <input type="text" class="form-control" name="modDate" th:value="${#temporals.format(dto.modDate, 'yyyy/MM/dd HH:mm:ss')}" readonly>
        </div>

        <a th:href="@{/guestbook/modify(gno = ${dto.gno}, page=${requestDTO.page}, type=${requestDTO.type}, keyword =${requestDTO.keyword})}">
            <button type="button" class="btn btn-primary">Modify</button>
        </a>

        <a th:href="@{/guestbook/list(page=${requestDTO.page} , type=${requestDTO.type}, keyword =${requestDTO.keyword})}">
            <button type="button" class="btn btn-info">List</button>
        </a>
```

<br/>

4. register.html

```html
		<form th:action="@{/guestbook/register}" th:method="post">
            <div class="form-group">
                <label >Title</label>
                <input type="text" class="form-control" name="title" placeholder="Enter Title">
            </div>
            <div class="form-group">
                <label >Content</label>
                <textarea class="form-control" rows="5" name="content"></textarea>
            </div>
            <div class="form-group">
                <label >Writer</label>
                <input type="text" class="form-control" name="writer" placeholder="Enter Writer">
            </div>

            <button type="submit" class="btn btn-primary">Submit</button>
        </form>
```