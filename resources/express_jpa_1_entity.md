
   프로젝트를 진행하던 중 구성원들 간의 기술스택에 있어서 차이점이 발생하였다.
   
   나는 mybatis던 jpa던 Entity(VO)클래스를 테이블과 거의 똑같이 만들었던 반면(1:N, M:N에서 각 클래스는 서로의 pk(id)를 참조하는 fk(id)만 가지고 있게),
   다른 분들은 각 클래스마다 참조 하는 클래스의 객체(또는 리스트)가 필드로 있었다.
   
   신기했던 것은 실제 DB테이블은 두 경우 모두 똑같았다.
   이 현상이 신기하였고 JPA를 공부해보고 싶은 마음이 있었기 때문에 공부를 하여 보겠다.
   콘솔에 Hibernate의 sql문을 분석하여 해당 방식에 문제가 있는지 등을 판단하는 데에도 도움이 될 것 같다.
   
   먼저 jpa를 오래 공부해온 팀내 선구자의 모든 소스를 받았고 이것들을 익히려고 한다.
   
<br/>

> 🚀 code

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseTimeEntity {

	@CreatedDate
	@Column(updatable = false)
	private LocalDateTime createdDate;

	@LastModifiedDate
	private LocalDateTime lastModifiedDate;
}
```

   기존에 알던 것이라 다행이다.
   1. @MappedSu~는 테이블로 생성하지 말라는 어노테이션이다.
   2. @~Date는 생성일시를 자동으로 채워준다.
   3. modifiedDate는 따로 만들어 유지할 것이기 때문에 createDate의 updateable은 false이다.
   
<br/>

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
@Getter
public class BaseEntity extends BaseTimeEntity{

	@CreatedBy
	@Column(updatable = false)
	private String createdBy;

	@LastModifiedBy
	private String lastModifiedBy;
}
```

   생소한 부분이다.
   조금 찾아보니 작성자와 수정자(사람!)에 대한 내용이었을 뿐이었다.
   위 클래스를 상속받았으므로 현재까지 4개의 필드가 있는 셈이다.(별도 테이블로는 생성되지 않는)
   
   <br/>

```java
@MappedSuperclass
@Getter
public class JpaBaseEntity {

	@Column(updatable = false)
	private LocalDateTime createdDate;
	private LocalDateTime updatedDate;

	@PrePersist
	public void prePersist() {
		LocalDateTime now = LocalDateTime.now();
		createdDate = now;
		updatedDate = now;
	}

	@PreUpdate
	public void preUpdate() {
		updatedDate = LocalDateTime.now();
	}
}
```

   또 다른 클래스이다.
   대충 예상을 해보자면 위 BaseTimeEntity와 똑같이 동작할 것 같다.
   @PrePersist가 생성될 때의 시간을 생성일자와 수정일자에 넣어줄 것 같다.
   @PreUpdate는 수정될 때 그 시간을 넣어줄 것 같다.
   
   BaseTimeEntity와의 차이는 @EntityListeners가 빠짐에 있다.
   찾아보니 @EntityListeners는 변경감지 알림에 해당한다.
   
   @Pre~어노테이션도 찾아보니 값을 넣어준다기 보다는 Entity가 생성, 수정되기 '직전' 시점으로 가는 것을 의미한다.
   그때 수동적으로 값을 넣어주는 로직을 넣어놨으니 결과적으로는 BaseTimeEntity와 같다.
   
<br/>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "name"})
public class Team extends JpaBaseEntity{

	@Id
	@GeneratedValue
	@Column(name = "team_id")
	private Long id;
	private String name;

	@OneToMany(mappedBy = "team")
	private List<Member> members = new ArrayList<>();

	public Team(String name) {
		this.name = name;
	}
}
```

   바로 위 JpaBaseEntity를 상속받은 Team클래스이다.
   @Entity로 db에 테이블로 생성된다.
   id(tead_id)는 @Id로 인해 pk이고 @Generated~로 인해 자동으로 값이 채워진다.
   
   한 멤버(뒤에서 만들 클래스)는 한 팀에만 소속되어 있고
   한 팀에는 여러명이 있다. (tbl_member) ∃- (tbl_team)이다.
   팀 쪽이 1이므로 @OneToMany이다.
   mappedBy속성은 '주인이 아닌'쪽에 선언하여 jpa에 알려주는 것이라고 한다.
   
   member가 team을 참조함으로(team테이블에는 member를 가리킬 필요 없다)
   mappedBy의 'team'은 멤버클래스에서 선언될 객체이름이다.
   
   n인쪽의 객체리스트를 가지고 있는 것 유념하자!
   Team클래스는 필드에 적힌 것 외에 생성/수정날짜를 가지고 있다.
   
<br/>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "username", "age"})
@NamedQuery(
	name = "Member.findByUsername",
	query="select m from Member m where m.username = :username"
)
@NamedEntityGraph(name = "Member.all", attributeNodes = @NamedAttributeNode("team"))
public class Member extends BaseEntity {

	@Id
	@GeneratedValue
	@Column(name = "member_id")
	private Long id;
	private String username;
	private int age;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "team_id")
	private Team team;

	public Member(String username) {
		this.username = username;
	}

	public Member(String username, int age, Team team) {
		this.username = username;
		this.age = age;
		if (team != null) {
			changeTeam(team);
		}
	}

	public Member(String username, int age) {
		this.username = username;
		this.age = age;
	}

	public void changeTeam(Team team) {
		this.team = team;
		team.getMembers().add(this);
	}
}
```
   대망의 멤버클래스이다.
   필드개수는 id, name, age, 상속받은4개, fk까지 총 8개이다.
   
   @NamedQuery는 정적 쿼리를 미리 만들어 놓는 어노테이션이다.
   미리 파싱하여 재사용할 수 있기 때문에 오류체크 및 조회 성능 향상에 도움이 될 수 있다.
   
   @NamedEntityGraph 는 연관된 엔티티를 조회할 때 사용하는 어노테이션이다.
   그래프 / 페치정책(lazy, eager) 등의 개념은 차후에 학습하겠다.
   
   일대다에서 1(주인아님)에 해당하는 부분은 mappedBy를 이용하였지만
   주인(n)에 해당하는 부분은 @JoinColumn으로 fk(id)값을 명시하여주면 된다.
   
   chageTeam메서드 : 팀(객체)을 바꾸고, 팀을 추가해준다.