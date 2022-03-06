> 🚀 1:1

![](https://images.velog.io/images/sonchanwoo/post/5cf9888b-4687-48ed-ae6a-b9b0a13268e8/image.png)

- fk가 있는 쪽이 '자식'이다.

- join선언과 이름선언은 자식이 한다.

```java
@Entity(name = "member_thumbnail")
@Getter
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class MemberThumbnail extends BaseEntity {

	@Id
	@GeneratedValue
	@Column(name = "member_thumbnail_id")
	private Long id;
	private String uuid;
	private String name;
	
	@OneToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "member_id")
	@JsonIgnore
	private Member member;
}
```

- 부모쪽에서 mappedBy를 한다.

```java
@Entity
@Getter
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member extends BaseEntity {

	@Id
	@GeneratedValue
	@Column(name = "member_id")
	private Long id;

	//생략

	@OneToOne(mappedBy = "member")
	private MemberThumbnail memberThumbnail;
}
```

<br/>

> 🚀 1:N

![](https://images.velog.io/images/sonchanwoo/post/b93a4bd2-d95e-4ba0-b49d-75c2f86d9d83/image.png)

- '1포스트'는 '1멤바' 가질 수 있다.

-> 멤버쪽은 1

- '1멤바는' 'n포스트' 가질 수 있다.

-> post쪽은 n

- 왼쪽으로 까마귀 발 이다.

-> 1:N에서 까마귀발 방향이 고민될 경우 '부모는 여러 자식을 가진다'를 떠올리면 쉽다.

```java
public class Member extends BaseEntity {

	@Id
	@GeneratedValue
	@Column(name = "member_id")
	private Long id;
    
    //생략..

	@OneToMany(mappedBy = "member")
	@JsonIgnore
	private List<Post> posts = new ArrayList<>();
}
```

-> 멤버가 부모 이므로 mappedBy를 가진다.

-> 여러 자식을 가지므로 List이다.

-> 부모(1)이므로 OneToMany

```java
public class Post {

	@Id
	@GeneratedValue
	@Column(name = "post_id")
	private Long id;
	private String title;
	private String introduce;
	private String content;
	private int likeCount;
	private int replyCount;
	private int viewCount;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "member_id")
	private Member member;

	//생략
}
```

- 자식이 조인한다.(위와 같음, 부모가 mappedBy)

- 자식은 한 부모이므로 객체 하나만 가진다.

- ManyToOne이다.(까마귀 발 방향을 생각하면 쉽다)

<br/>

> 🚀 N:M

- 다대다는 양쪽 테이블 모두를 부모로 하며, 중간에 바람난 자식이 생긴다.

![](https://images.velog.io/images/sonchanwoo/post/cb3c0182-5134-4124-a0aa-c8f6174d8293/image.png)

```java
public class Post {

	//생략

	@ManyToMany(mappedBy = "posts")
	private List<Tag> tags = new ArrayList<>();

	//생략
}
```

```java
public class Tag extends BaseEntity {

	@ManyToMany
	@JoinTable(name = "post_tag",
		joinColumns = @JoinColumn(name = "tag_id"),
		inverseJoinColumns = @JoinColumn(name = "post_id"))
	@JsonIgnore
	private List<Post> posts = new ArrayList<>();
}
```

- 양쪽 다 부모 이므로 List로 갖는 다는 것과 manyTomany는 알겠다.

- 자식이 joinColumn, 부모가 mappedBy는 무조건은 아니었다 보다.

- ManyToMany가 아닌 OneToMany + ManyToOne의 경우를 아래에서 보겠다.

<br/>

> 🚀 N:M(일대다 두개를 통한)

- 특정 사용자가 어떤 게시글을 읽었는지, 특정 게시글은 누구에게 읽혔는지

```java
	//Post 엔티티
	@OneToMany(mappedBy = "post")
	@JsonIgnore
	private List<ReadPost> readPosts = new ArrayList<>();
```

```java
	//Member 엔티티    
    @OneToMany(mappedBy = "member")
	@JsonIgnore
	private List<ReadPost> readPosts = new ArrayList<>();
```

```java
	//ReadPost 엔티티
	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "post_id")
	private Post post;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "member_id")
	private Member member;
    
```

- @ManyToMany와 다르게 아래의 규칙들이 모두 지켜진다.

<br/>

> 🚀 헷갈리지 않게 규칙 정리

1. 부모가 mappedBy

2. 자식이 @JoinColumn(name)

3. 부모(많이 가질 수 있다)쪽이 아닌 자식(하나의 부모)쪽이 까마귀 발이다.

4. 까마귀 발과 OneToMany 등의 어노테이션은 상식적으로 맞춰진다.