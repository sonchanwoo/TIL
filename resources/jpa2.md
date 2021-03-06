> ๐ ํ์ด์ง ์ฒ๋ฆฌ

- ํ์คํธ ๋ฉ์๋ - selectAllWithPage

```java
    @Test
    public void testPageDefault(){
        Pageable pageable = PageRequest.of(0,10);

        Page<Memo> result = memoRepository.findAll(pageable);
    }
```

1. Pageable

- pageable์ ์ธํฐํ์ด์ค ์ด๊ธฐ ๋๋ฌธ์ ๊ตฌํ์ฒด์ธ PageRequest์ static ๋ฉ์๋์ธ of( )์ผ๋ก ์ฒ๋ฆฌํ๋ค.
  
  - of(int page, int size)
    
  - of(int page, int size, Sort sort)
    
- (0,10)์ด๋ 1ํ์ด์ง์ 10๊ฐ๋ฅผ ์๋ฏธํ๋ค. (0-indexed)

- Pageable์ ์ฌ์ฉํ๋ ์ฟผ๋ฆฌ๋ Pageํ์์ ๋ฐํํ๋ค.
  
<br/>
  
2. findAll( )

- findAll() : selectAll
  
- findAll(Sort sort) : selectAll + order by (asc/desc)
  
- findAll(Pageable pageable) : selectAll + limit/count
  
- ํ์ด์ง ์ฒ๋ฆฌ์ ํ์ํ limit์ count๋ฅผ ๋ด๋ถ์ ์ผ๋ก ์ฌ์ฉ
  
```
Hibernate: 
    select
        memo0_.mno as mno1_0_,
        memo0_.memo_text as memo_tex2_0_ 
    from
        tbl_memo memo0_ limit ?
Hibernate: 
    select
        count(memo0_.mno) as col_0_0_ 
    from
        tbl_memo memo0_
```
  
- ๋ฐํ ํ์์ด List๊ฐ ์๋๋ผ Page๋ผ๋ ์   

  - List + ๋ถ๊ฐ๊ธฐ๋ฅ(์๋)
  
![](https://images.velog.io/images/sonchanwoo/post/dbb6334f-06e3-4509-a27d-81258cfbaba2/image.png)

<br/>

> ๐ ์ ๋ ฌ ์กฐ๊ฑด ์ถ๊ฐ

```java
    @Test
    public void testSort(){
        Sort sort1 = Sort.by("mno").descending();
        Sort sort2 = Sort.by("memoText").ascending();
        Sort sortAll = sort1.and(sort2);

        Pageable pageable = PageRequest.of(0,10, sortAll);

        Page<Memo> result = memoRepository.findAll(pageable);
    }
```

1. Sort ์ฌ์ฉ๋ฒ(by, ~scending(), and) ์ฃผ๋ชฉ

2. ์ค๋ฒ๋ผ์ด๋ฉ ๋ of ๋ฉ์๋ ์ฌ์ฉ(์์์ ์ค๋ช)

3. ๋ก๊ทธ

```
Hibernate: 
    select
        memo0_.mno as mno1_0_,
        memo0_.memo_text as memo_tex2_0_ 
    from
        tbl_memo memo0_ 
    order by
        memo0_.mno desc,
        memo0_.memo_text asc limit ?
Hibernate: 
    select
        count(memo0_.mno) as col_0_0_ 
    from
        tbl_memo memo0_
```

<br/>

> ๐ Query Methods
- ์ฟผ๋ฆฌ๋ฉ์๋๋ ๋ฉ์๋์ ์ด๋ฆ ์์ฒด๊ฐ ์ง์๋ฌธ์ด ๋๋ ๊ธฐ๋ฅ์ด๋ค.
- ์์ ์์ ๋ค์ด JpaRepository๊ฐ ๊ธฐ๋ณธ์ ์ผ๋ก ์ ๊ณตํ๋ฉฐ, ์๋์์ฑ์ผ๋ก ๋ณด์ด๋ ๋ฉ์๋๋ฅผ ์ฌ์ฉํ์๋ค๋ฉด
- ์ฟผ๋ฆฌ ๋ฉ์๋๋ ๊ธฐ๋ณธ์ ์ผ๋ก ์๊ธด ์์ผ๋ฉด์๋ ์ฌ์ค ์๋ ๋ฉ์๋๋ค์, ๋ฉ์๋ ์ด๋ฆ์ ์ ์ง์ด์ + ์ ์ฐพ์์ ์ฌ์ฉํ๋ ๊ฒ์ด๋ค.

```java
public interface MemoRepository  extends JpaRepository<Memo,Long> {

    //1
    List<Memo> findByMnoBetweenOrderByMnoDesc(Long from,Long to);
    
    //2
    Page<Memo> findByMnoBetween(Long from, Long to, Pageable pageable);
}
```

1. ๋ ํผ๋ฐ์ค๋ฅผ ์ ์ฐพ๊ฑฐ๋, ์ฟผ๋ฆฌ ๋ฉ์๋๋ฅผ ์ํ ์ธํ๋ฆฌ์ ์ด์ ์๋์์ฑ์ ๋์์ ๋ฐ์, ๊ท์น์ ๋ง๋(๋ช๋ช๋ฒ, ๋งค๊ฐ๋ณ์, ๋ฐํํ์ ๋ฑ) ๋ฉ์๋๋ฅผ ์ ์ฐพ์์ ์ฌ์ฉํ  ์ ์๋ค.

2. Pageable์ ๋๋ถ๋ถ ์ด๋ ์ฟผ๋ฆฌ๋ฉ์๋์๋ ๊ฒฐํฉํ  ์ ์๊ธฐ ๋๋ฌธ์ ๋ณดํต ํ์ด์ง๊ณผ ์ ๋ ฌ์กฐ๊ฑด์ ๋นผ๊ณ ,  1->2 ์ฒ๋ผ ๋ง๋ ๋ค.

3. sql์ด ๋ณต์กํด์ง์๋ก ๋ถํธํด์ง๊ธฐ ๋๋ฌธ์, ๊ฐ๋จํ ์ฒ๋ฆฌ๋ง ์ฟผ๋ฆฌ๋ฉ์๋๋ฅผ ์ด์ฉํ๊ณ , ๋ค์ ํ ํฝ์ด ๊ฐ์ฅ ์ผ๋ฐ์ ์ด๋ค.

<br/>

> ๐ @Query, JPQL(๊ฐ์ฒด์งํฅ ์ฟผ๋ฆฌ)
- ORM์ ๊ฐ๋์ ์ ์ฉํ๋ค๋ ๋๋์ด ํ ๋ ๋ค. (Table -> Class, instance -> row, column -> field)

1. 

```java
    @Query("select memo from Memo memo order by memo.mno desc")
    List<Memo> getListDesc();
```

```
Hibernate: 
    select
        memo0_.mno as mno1_0_,
        memo0_.memo_text as memo_tex2_0_ 
    from
        tbl_memo memo0_ 
    order by
        memo0_.mno desc
```

<br/>

2. 

```java
    //ํ๋ผ๋ฏธํฐ๊ฐ ํ๋(์ปฌ๋ผ)์ผ ๋ :xxx ์ผ๋ก !
    //pageable์ ์ค์ ํ ๊ฑฐ๋ฉด countQuery์์ฑ ์ ์ฉ!
    @Query(value = "select memo from Memo memo where memo.mno > :mno", countQuery = "select count(memo) from Memo memo where memo.mno > :mno ")
    Page<Memo> getListWithQuery(Long mno, Pageable pageable);
```

```
Hibernate: 
    select
        memo0_.mno as mno1_0_,
        memo0_.memo_text as memo_tex2_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno>? 
    order by
        memo0_.mno desc limit ?

Hibernate: 
    select
        count(memo0_.mno) as col_0_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno>?
```


<br/>

3.

```java
    //ํ๋ผ๋ฏธํฐ๊ฐ ํ๋(์ปฌ๋ผ)์ผ ๋ :xxx ์ผ๋ก !
    //delete, update, insert dml(select ์ ์ธ)๋ ๋ ์ด๋ธํ์ด์์ ๋ถ์ฌ์ค์ผ ํ๋ค.
    @Query("update Memo memo set memo.memoText = :memoText where memo.mno = :mno")
    @Transactional
    @Modifying
    int updateMemoText(@Param("mno") Long mno, @Param("memoText") String memoText);
```

```
Hibernate: 
    update
        tbl_memo 
    set
        memo_text=? 
    where
        mno=?
```


<br/>

4.

```java
    //ํ๋ผ๋ฏธํฐ๊ฐ ๊ฐ์ฒด(์ํฐํฐ)์ผ ๋ : :#{} ์ผ๋ก !
    @Query("update Memo memo set memo.memoText = :#{#param.memoText} where memo.mno = :#{#param.mno}")
    @Transactional
    @Modifying
    int updateMemoText(@Param("param") Memo memo );
```

```
Hibernate: 
    update
        tbl_memo 
    set
        memo_text=? 
    where
        mno=?
```


<br/>

5.

```java
    //JPQL์ ์ฅ์ ์ Object๋ ๋ฐ์์ฌ ์ ์์
    //๋์ค์ ์กฐ์ธ ๋ฑ์ผ๋ก ์ ๋นํ ์ํฐํฐํ์์ด ์กด์ฌํ์ง ์์ ๊ฒฝ์ฐ์
    @Query("select memo.mno, memo.memoText, current_date from  Memo memo where memo.mno > :mno")
    List<Object[]> getListWithQueryObject(Long mno);
```

```
Hibernate: 
    select
        memo0_.mno as col_0_0_,
        memo0_.memo_text as col_1_0_,
        current_date as col_2_0_ 
    from
        tbl_memo memo0_ 
    where
        memo0_.mno>?
```


<br/>

6.

```java
    //์ด์ฉ ์ ์๋ ๊ฒฝ์ฐ์ ์ฌ์ฉํ  Native SQL๋ ์์ผ๋ฉฐ, nativeQuery๋ฅผ true๋ก ๋๊ณ  ์ฌ์ฉํ๋ค.
    @Query(value="select * from tbl_memo where mno > 0", nativeQuery = true)
    List<Object[]> getNativeResult();
```

```
Hibernate: 
    select
        * 
    from
        tbl_memo 
    where
        mno > 0
```


<br/>