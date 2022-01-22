> 🚀 페이징 처리

- 테스트 메서드 - selectAllWithPage

```java
    @Test
    public void testPageDefault(){
        Pageable pageable = PageRequest.of(0,10);

        Page<Memo> result = memoRepository.findAll(pageable);
    }
```

1. Pageable

- pageable은 인터페이스 이기 때문에 구현체인 PageRequest의 static 메서드인 of( )으로 처리한다.
  
  - of(int page, int size)
    
  - of(int page, int size, Sort sort)
    
- (0,10)이란 1페이지의 10개를 의미한다. (0-indexed)

- Pageable을 사용하는 쿼리는 Page타입을 반환한다.
  
<br/>
  
2. findAll( )

- findAll() : selectAll
  
- findAll(Sort sort) : selectAll + order by (asc/desc)
  
- findAll(Pageable pageable) : selectAll + limit/count
  
- 페이징 처리에 필요한 limit와 count를 내부적으로 사용
  
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
  
- 반환 타입이 List가 아니라 Page라는 점  

  - List + 부가기능(아래)
  
![](https://images.velog.io/images/sonchanwoo/post/dbb6334f-06e3-4509-a27d-81258cfbaba2/image.png)

<br/>

> 🚀 정렬 조건 추가

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

1. Sort 사용법(by, ~scending(), and) 주목

2. 오버라이딩 된 of 메서드 사용(위에서 설명)

3. 로그

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

> 🚀 Query Methods
- 쿼리메서드란 메서드의 이름 자체가 질의문이 되는 기능이다.
- 위의 예제들이 JpaRepository가 기본적으로 제공하며, 자동완성으로 보이는 메서드를 사용하였다면
- 쿼리 메서드란 기본적으로 있긴 있으면서도 사실 없는 메서드들을, 메서드 이름을 잘 지어서 + 잘 찾아서 사용하는 것이다.

```java
public interface MemoRepository  extends JpaRepository<Memo,Long> {

    //1
    List<Memo> findByMnoBetweenOrderByMnoDesc(Long from,Long to);
    
    //2
    Page<Memo> findByMnoBetween(Long from, Long to, Pageable pageable);
}
```

1. 레퍼런스를 잘 찾거나, 쿼리 메서드를 위한 인텔리제이의 자동완성의 도움을 받아, 규칙에 맞는(명명법, 매개변수, 반환타입 등) 메서드를 잘 찾아서 사용할 수 있다.

2. Pageable은 대부분 어느 쿼리메서드에도 결합할 수 있기 때문에 보통 페이징과 정렬조건을 빼고,  1->2 처럼 만든다.

3. sql이 복잡해질수록 불편해지기 때문에, 간단한 처리만 쿼리메서드를 이용하고, 다음 토픽이 가장 일반적이다.

<br/>

> 🚀 @Query, JPQL(객체지향 쿼리)
- ORM의 개념을 적용했다는 느낌이 확 든다. (Table -> Class, instance -> row, column -> field)

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
    //파라미터가 필드(컬럼)일 때 :xxx 으로 !
    //pageable을 줘서 할거면 countQuery속성 적용!
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
    //파라미터가 필드(컬럼)일 때 :xxx 으로 !
    //delete, update, insert dml(select 제외)는 두 어노테이션을 붙여줘야 한다.
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
    //파라미터가 객체(엔티티)일 때 : :#{} 으로 !
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
    //JPQL의 장점은 Object도 받아올 수 있음
    //나중에 조인 등으로 적당한 엔티티타입이 존재하지 않을 경우에
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
    //어쩔 수 없는 경우에 사용할 Native SQL도 있으며, nativeQuery를 true로 놓고 사용한다.
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