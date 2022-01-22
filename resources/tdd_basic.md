## @ before & after

```java
public class EachTests {
    @BeforeEach
    public void setUp(){
        System.out.println("setUp");
    }

    @Test
    void test1(){
        System.out.println("test1");
    }

    @Test
    void test2(){
        System.out.println("test2");
    }

    @AfterEach()
    void tearDown(){
        System.out.println("tearDown");
    }
}
```

---

### - test1() 실행 시

    setUp
    test1
    tearDown
    
---

### - test1(), test2() 모두 실행 시

    setUp
    test1
    tearDown
    
    setUp
    test2
    tearDown
    
---

<br/>
<br/>

## @ assertThat()

`아래 모든 test들은 pass한다.`

```java
    @Test
    void test(){

        //String
        assertThat("").isEqualTo("");
        assertThat("").contains();

        //int
        assertThat(1).isEqualTo(1);
        assertThat(1).isEven();

        //char
        assertThat('d').isEqualTo('2');
        assertThat('d').inUnicode();

        //boolean
        assertThat(true).isEqualTo(true);
        assertThat(true).isTrue();
        assertThat(false).isFalse();

    }
```

- assertThat()의 매개변수 타입에 따라 이용 가능한 메서드가 상이하다.
- 필요할 때마다 찾아서 사용하면 되겠다.

---
### - 여러개일 경우

```java
    @Test
    void test1(){
        String[] arr ={"abc","aab","vvs"};
        
        assertThat(arr).contains("abc");
        assertThat(arr).contains("abc", "aab");
        assertThat(arr).contains("abc", "aab", "aab");
        assertThat(arr).contains("vvs");

        assertThat(arr).containsExactly("abc","aab","vvs");
    }
```


- contains는 말 그대로 포함하는가의 의미로써 정말 있기만 하면 된다.
  - 당연히 일부의 포함여부 가능하다.
  - 순서 상관없다.
  - 중복도 가능하다.
  
- containsExactly는 "정말 딱 이거냐"의 의미이다.
  - 순서 상관있다.
  
- List도 동일하게 사용 가능하다.

---


### - 여러개일 경우 - 2 (ParameterizedTest)

```java
    @Test
    void test(){
        int[] arr = {1,2,3};
        assertThat(arr).contains(1);
        assertThat(arr).contains(2);
        assertThat(arr).contains(3);
    }
```

이 코드는

```java
    @ParameterizedTest
    @ValueSource(ints={1,2,3})
    void test(int value){
        int[] arr = {1,2,3};
        assertThat(arr).contains(value);
    }
```

이렇게 가능하다.

<br/>

결과 값을 다르게 주고 싶다면 (실패케이스도 pass시키기 위해)

```java
    @ParameterizedTest
    @CsvSource(value = {"1:true","2:true","3:true","4:false","5:false"}, delimiter = ':')
    void test(int input, boolean output){
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        assertThat(list.contains(input)).isEqualTo(output);
    }
```

이렇게 가능하며, value값이 파라미터의 타입에 따라 자동으로 매핑된다.


---

### - exception 다루기

```java
    void test4(){
        String arr = "";

        //방법 - 1
        assertThatThrownBy(() -> {
            arr.charAt(1);
        }).isInstanceOf(IndexOutOfBoundsException.class);
        
        //방법 - 2
        assertThatExceptionOfType(IndexOutOfBoundsException.class)
                .isThrownBy(()->{
                    arr.charAt(1);
                });
    }
```

```java
    @Test
    void test4(){        
        String s=null;
        //자주 사용되는 예외는(IO, IllegalState, IllegalArgument, NullPointer ...)
        //만들어져 있는 것을 사용하면 된다.
        assertThatNullPointerException().isThrownBy(()->{s.length();});
    }
```