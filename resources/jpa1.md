> 🚀 ORM과 JPA

- 클래스와 테이블이, 인스턴스와 row(record-tuple)가 유사한 점에서 시작한다.
- 객체지향과 관계형 데이터베이스의 매핑 패러다임이다.
- ORM을 java에서 사용하기 위한 스펙이 JPA이다.
- JPA 스펙의 여러 프레임워크들 중 spring boot는 Hiberbate 구현체를 이용한다.

<br/>

> 🚀 엔티티 클래스

```java
package org.zerock.ex2.entity;

@Entity
@Table(name = "tbl_memo")
@ToString
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Memo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mno;

    @Column(length = 200,nullable = false)    
    private String memoText;
}
```

- @Entity

엔티티를 위한 클래스, JPA로 관리되는 인스턴스이려면 반드시 추가해야하는 어노테이션이다.

<br/>

- @Table

@Entity와 함께 사용되어 테이블 이름, 인덱스 등의 설정을 하는 어노테이션이다.

<br/>

- @Id : PK에 해당하며, entity클래스라면 꼭 있어야 한다.
- @GeneratedValue(strategy = GenerationType.IDENTITY)

pk 값을 자동으로 생성하고자 할 때 어떤 '키 생성 전략'으로 생성할 것인지를 결정한다.

IDENTITY : 해당하는 데이터베이스가 키생성을 결정하는 것이다.(mysql & mariadb는 auto_increment)

<br/>

- @Column(length = 200,nullable = false)

칼럼에 정보를 제공한다(기본값,null,길이,이름 등)

- @Transient : 테이블에 컬럼으로 생성하지 않고 싶은 필드에 사용

<br/>

> 🚀 JpaRepository

- JpaRepository를 상속하는 인터페이스를 작성만 하여도 스프링의 빈으로 등록된다.(Non-scan)

```java
package org.zerock.ex2.repository;

public interface MemoRepository  extends JpaRepository<Memo,Long> {
//엔티티타입과 @Id의 타입을 지정
}
```

<br/>

- 테스트 클래스

```java
@SpringBootTest
public class MemoRepositoryTests {

    @Autowired
    MemoRepository memoRepository;
    
    //아래 테스트 메서드들은 이 부분에 삽입될 예정.    
    
}
```

<br/>

- 테스트 메서드 - insert

```java
    @Test
    public void testInsertDummies(){
        IntStream.rangeClosed(1,10).forEach(i->{
            Memo memo = Memo.builder().memoText("sample..."+i).build();
            memoRepository.save(memo);
        });
    }
```

1. save( )는 없으면 insert, 있으면 update로 동작한다.
2. 로그에서 Hibernate가 발생시키는 구문을 확인하며 공부할 것.

<br/>

- 테스트 메서드 - select

```java
    @Test
    public void testSelect(){
        Long mno = 10L;

        Optional<Memo> result = memoRepository.findById(mno);

        if(result.isPresent()){
            Memo memo = result.get();
            System.out.println(memo);
        }
    }
```

1. null 체크를 포함하기 위해 Optional 타입으로 나온다.

2. getOne( )과 비교하려고 했지만 getOne이 deprecated라서 제외하였다.

<br/>

- 테스트 메서드 - update

```java
    @Test
    public void testUpdate(){
        Memo memo = Memo.builder().mno(10L).memoText("update...").build();
        System.out.println(memoRepository.save(memo));      
    }
```

1. 스키마에 이미 존재하는 @Id를 주면 save는 update로 동작한다.

```
Hibernate: 
    select
        memo0_.mno as mno1_0_0_,
        memo0_.memo_text as memo_tex2_0_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno=?
Hibernate: 
    update
        tbl_memo 
    set
        memo_text=? 
    where
        mno=?
```

2. 없는 값을 주고 실행하면 insert로 동작한다.

```
Hibernate: 
    select
        memo0_.mno as mno1_0_0_,
        memo0_.memo_text as memo_tex2_0_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno=?
Hibernate: 
    insert 
    into
        tbl_memo
        (memo_text) 
    values
        (?)
```

<br/>

- 테스트 메서드 - delete
  - 없으면 에러를 발생시켜 버린다...

```java
    @Test
    public void testDelete(){
        Long mno = 11l;

        memoRepository.deleteById(mno);
    }
```

1. 있는 @Id

```
Hibernate: 
    select
        memo0_.mno as mno1_0_0_,
        memo0_.memo_text as memo_tex2_0_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno=?
Hibernate: 
    delete 
    from
        tbl_memo 
    where
        mno=?
```

2. 없는 @Id

```
Hibernate: 
    select
        memo0_.mno as mno1_0_0_,
        memo0_.memo_text as memo_tex2_0_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno=?

No class org.zerock.ex2.entity.Memo entity with id 10 exists!
```