> ๐ ORM๊ณผ JPA

- ํด๋์ค์ ํ์ด๋ธ์ด, ์ธ์คํด์ค์ row(record-tuple)๊ฐ ์ ์ฌํ ์ ์์ ์์ํ๋ค.
- ๊ฐ์ฒด์งํฅ๊ณผ ๊ด๊ณํ ๋ฐ์ดํฐ๋ฒ ์ด์ค์ ๋งคํ ํจ๋ฌ๋ค์์ด๋ค.
- ORM์ java์์ ์ฌ์ฉํ๊ธฐ ์ํ ์คํ์ด JPA์ด๋ค.
- JPA ์คํ์ ์ฌ๋ฌ ํ๋ ์์ํฌ๋ค ์ค spring boot๋ Hiberbate ๊ตฌํ์ฒด๋ฅผ ์ด์ฉํ๋ค.

<br/>

> ๐ ์ํฐํฐ ํด๋์ค

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

์ํฐํฐ๋ฅผ ์ํ ํด๋์ค, JPA๋ก ๊ด๋ฆฌ๋๋ ์ธ์คํด์ค์ด๋ ค๋ฉด ๋ฐ๋์ ์ถ๊ฐํด์ผํ๋ ์ด๋ธํ์ด์์ด๋ค.

<br/>

- @Table

@Entity์ ํจ๊ป ์ฌ์ฉ๋์ด ํ์ด๋ธ ์ด๋ฆ, ์ธ๋ฑ์ค ๋ฑ์ ์ค์ ์ ํ๋ ์ด๋ธํ์ด์์ด๋ค.

<br/>

- @Id : PK์ ํด๋นํ๋ฉฐ, entityํด๋์ค๋ผ๋ฉด ๊ผญ ์์ด์ผ ํ๋ค.
- @GeneratedValue(strategy = GenerationType.IDENTITY)

pk ๊ฐ์ ์๋์ผ๋ก ์์ฑํ๊ณ ์ ํ  ๋ ์ด๋ค 'ํค ์์ฑ ์ ๋ต'์ผ๋ก ์์ฑํ  ๊ฒ์ธ์ง๋ฅผ ๊ฒฐ์ ํ๋ค.

IDENTITY : ํด๋นํ๋ ๋ฐ์ดํฐ๋ฒ ์ด์ค๊ฐ ํค์์ฑ์ ๊ฒฐ์ ํ๋ ๊ฒ์ด๋ค.(mysql & mariadb๋ auto_increment)

<br/>

- @Column(length = 200,nullable = false)

์นผ๋ผ์ ์ ๋ณด๋ฅผ ์ ๊ณตํ๋ค(๊ธฐ๋ณธ๊ฐ,null,๊ธธ์ด,์ด๋ฆ ๋ฑ)

- @Transient : ํ์ด๋ธ์ ์ปฌ๋ผ์ผ๋ก ์์ฑํ์ง ์๊ณ  ์ถ์ ํ๋์ ์ฌ์ฉ

<br/>

> ๐ JpaRepository

- JpaRepository๋ฅผ ์์ํ๋ ์ธํฐํ์ด์ค๋ฅผ ์์ฑ๋ง ํ์ฌ๋ ์คํ๋ง์ ๋น์ผ๋ก ๋ฑ๋ก๋๋ค.(Non-scan)

```java
package org.zerock.ex2.repository;

public interface MemoRepository  extends JpaRepository<Memo,Long> {
//์ํฐํฐํ์๊ณผ @Id์ ํ์์ ์ง์ 
}
```

<br/>

- ํ์คํธ ํด๋์ค

```java
@SpringBootTest
public class MemoRepositoryTests {

    @Autowired
    MemoRepository memoRepository;
    
    //์๋ ํ์คํธ ๋ฉ์๋๋ค์ ์ด ๋ถ๋ถ์ ์ฝ์๋  ์์ .    
    
}
```

<br/>

- ํ์คํธ ๋ฉ์๋ - insert

```java
    @Test
    public void testInsertDummies(){
        IntStream.rangeClosed(1,10).forEach(i->{
            Memo memo = Memo.builder().memoText("sample..."+i).build();
            memoRepository.save(memo);
        });
    }
```

1. save( )๋ ์์ผ๋ฉด insert, ์์ผ๋ฉด update๋ก ๋์ํ๋ค.
2. ๋ก๊ทธ์์ Hibernate๊ฐ ๋ฐ์์ํค๋ ๊ตฌ๋ฌธ์ ํ์ธํ๋ฉฐ ๊ณต๋ถํ  ๊ฒ.

<br/>

- ํ์คํธ ๋ฉ์๋ - select

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

1. null ์ฒดํฌ๋ฅผ ํฌํจํ๊ธฐ ์ํด Optional ํ์์ผ๋ก ๋์จ๋ค.

2. getOne( )๊ณผ ๋น๊ตํ๋ ค๊ณ  ํ์ง๋ง getOne์ด deprecated๋ผ์ ์ ์ธํ์๋ค.

<br/>

- ํ์คํธ ๋ฉ์๋ - update

```java
    @Test
    public void testUpdate(){
        Memo memo = Memo.builder().mno(10L).memoText("update...").build();
        System.out.println(memoRepository.save(memo));      
    }
```

1. ์คํค๋ง์ ์ด๋ฏธ ์กด์ฌํ๋ @Id๋ฅผ ์ฃผ๋ฉด save๋ update๋ก ๋์ํ๋ค.

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

2. ์๋ ๊ฐ์ ์ฃผ๊ณ  ์คํํ๋ฉด insert๋ก ๋์ํ๋ค.

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

- ํ์คํธ ๋ฉ์๋ - delete
  - ์์ผ๋ฉด ์๋ฌ๋ฅผ ๋ฐ์์์ผ ๋ฒ๋ฆฐ๋ค...

```java
    @Test
    public void testDelete(){
        Long mno = 11l;

        memoRepository.deleteById(mno);
    }
```

1. ์๋ @Id

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

2. ์๋ @Id

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