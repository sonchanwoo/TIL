<img src="https://images.velog.io/images/sonchanwoo/post/e6d72558-8c0b-43b4-9c51-407d0ec96371/20220107_151331.png" width="1000"/>


<img src="https://images.velog.io/images/sonchanwoo/post/3d919033-c04d-48cf-9946-a75144d14409/20220107_151348.png" width="1000"/>


<img src="https://images.velog.io/images/sonchanwoo/post/30ca1266-5045-4df7-9698-042e7b60338a/20220107_151429.png" width="1000"/>


갈피를 못잡아서 피드백 강의부터 보았다.

<br/>

***

> #### 🚀 강의의 내용은 이러했다
- 단위 테스트를 먼저 작성한 후, production 코드작성
	- 실패하는 단위테스트를 꼭 봐야함
- ToDoList 작성하여 단위테스트
- 작은 단위로 쪼개어 단위테스트
- 리팩토링
- 클래스 잘 나누기

"정답이 있다", "마스터하였다" 등의 개념이 아니라 끊임 없이 고민하는 것.

따라치려고 하는 것 보다는, 감 잡고 또 해보고.. 영상 또 보고.. 또 해보고..

이런 식으로 하려고 한다.

<br/>

---

> #### 🏀 To Do List
* 1 ~ 9까지의 서로 다른 수 3자리 생성(컴퓨터용)
* 1 ~ 9까지의 서로 다른 수 3자리 생성(사용자용)
* 볼 맞히기
  * 사용자 볼 하나 당...
  * ...
* 게임 종료하기
* ...

이런 식으로 큼지막한 ToDoList가 나왔다.

감은 잡았지만, 막상 하려니 막막했다.

<br/>

___

> #### 🏀 지우고 지우기를 반복했다.
- 아까 봤던 건 이게 아닌데..
- 메서드를 빼야할 일이 생겨( indent depth / 한 메서드는 한 일만 )
  빼다보니, 네이밍이 적절하지 않게 되는 문제가 많았다.
- 그냥 무언가 이상해...
* ...

이 과정을 반복하다가 '가장 작은 단위'로 쪼개야 편해진다는 걸 알았다.
(잘하고 있다고는 장담할 수 없지만..)
<br/>

---

> #### 🏀 사용자의 입력은
- 1~9 사이
- 3 자리
- 서로 다른 수


`-    -   -   -` : 아쉬운 점

---

🏀 1~9 사이

``` java
    @Test
    @DisplayName("사용자가 1~9사이의 숫자를 입력하였는지 체크")
    void test(){
        BallMaker ballMaker = new BallMaker();
        
        assertThat(ballMaker.checkRange(0)).isFalse();
        assertThat(ballMaker.checkRange(10)).isFalse();
        
        assertThat(ballMaker.checkRange(1)).isTrue();
        assertThat(ballMaker.checkRange(9)).isTrue();
    }
```

- 하나씩 1~9 범위 내에 있는지 체크했다.
- 0,10은 실패 경우의 시작점을 의미한다.
- 1,9는 성공 경우의 양끝이다.(다 할 필요는 없으니까)


`ParameterizedTest 를 이용하지 않음(테스트코드 리팩토링)`
`언밀하게는 한 숫자의 범위를 체크하는 것이니 이름이 조금..`
`DisPlayName을 이런식으로 사용하는 게 맞나..`

---

```java
public class BallMaker {
    private static final int MIN_NUM=1;
    private static final int MAX_NUM=9;
    private static final int DIGIT_NUM=3;
    //범위와 자릿수는 얼마든지 바뀔 수 있으니 빼두었다.

    public boolean checkRange(int i) {
        //return i>=1 && i<=9;
        return i>=MIN_NUM && i<=MAX_NUM;
    }
}
```

- 범위와 자릿수는 얼마든지 바뀔 수 있으니 빼두었!!으나...

`1부터(MIN_NUM) 9까지(MAX_NUM) 3자리(DIGIT_NUM)는 어찌보면 게임룰이니`
`BallMaker가 아니라 더 외부에(게임 자체와 관련된) 있어야 하는 것일까?`
`생성자를 통해서 field를 초기화 해야할까?`

---

🏀 3 자리수

```java
  @Test
  @DisplayName("사용자가 3자리수를 입력하였는지")
  void test2(){
      BallMaker ballMaker = new BallMaker();
      
      assertThat(ballMaker.checkNum(11)).isFalse();
      assertThat(ballMaker.checkNum(1111)).isFalse();
      
      assertThat(ballMaker.checkNum(111)).isTrue();
  }
```

* 3자리수만 허용함을 확인할 수 있다.

`BallMaker ballMaker = new BallMaker()는 공통이었구나.. @BeforeEach`
`어우.. ParameterizedTest 빨리 적용하던가 해야겠다`

---

```java
public boolean checkNum(int i) {
    //return i/100 != 0 && i/1000 == 0;
    return MyMathUtil.checkDigit(i,DIGIT_NUM);
}
```

- 게임 룰 중 자릿수는 변할 여지가 있으니
  i/100 != 0 && ... 방식은 적절하지 않다고 생각했다.
- 가만보면 이 논리는 BallMaker와는 어울리지 않고
  수학적인 일이니 따로 Util을 만들고자 하였다.
  (당연히 test코드 먼저 작성하였었다.)
  
---

- 123은 100으로 나누면 몫이 있고, 1000으로 나누면 몫이 없다.
- 이 두 논리를 나누었다.(나중에 괜한 짓이라는 것을 알았지만..)

```java
    @Test
    @DisplayName("n자리 이하 수라면 10^n로 나누면 몫이 0이야")
    void test3(){
        assertThat(MyMathUtil.checkDigitUpper(0,3)).isTrue();
        assertThat(MyMathUtil.checkDigitUpper(1,3)).isTrue();
        assertThat(MyMathUtil.checkDigitUpper(111,3)).isTrue();

        assertThat(MyMathUtil.checkDigitUpper(1111,3)).isFalse();
    }
```

`@DisPlayName을 이런식으로 사용해도 되나..? - 2`
`3자리 이하면 true, 4자리 부터는 false인데 어떻게 value를 짜야 적당한지 모르겠다.`

```java
    @Test
    @DisplayName("n자리 이상 수라면 10^(n-1)로 나누면 몫이 0이 아니야")
    void test4(){
        assertThat(MyMathUtil.checkDigitLower(11,3)).isFalse();

        assertThat(MyMathUtil.checkDigitLower(111,3)).isTrue();
        assertThat(MyMathUtil.checkDigitLower(1111,3)).isTrue();
    }
```

`이렇게 3개부터 true라면 values(2,3,4) 하면 될까..?`

```java
public class MyMathUtil {
    public static boolean checkDigitUpper(int i, int digit) {
        return i/(int)Math.pow(10,digit) == 0;
    }

    public static boolean checkDigitLower(int i, int digit) {
        return i/(int)Math.pow(10,digit-1) != 0;
    }
}
```

`이 메서드들은 먼 훗날 private이어야 함`
`private method를 reflection이용하여 테스트 하는 것까지는`
`강사님께서도 좀 그렇다는 뉘앙스`
`그럼 private로 바꾸고 나서는 테스트코드에 에러가 날텐데`
`테스트 통과 후 private로 바꾸면 그 테스트코드는 지우나 그럼..?`

---

- 두개를 합치다 !

```java
    @Test
    @DisplayName("위 두개를 합침")
    void test5(){
        assertThat(MyMathUtil.checkDigit(11,3)).isFalse();
        assertThat(MyMathUtil.checkDigit(1111,3)).isFalse();
        
        assertThat(MyMathUtil.checkDigit(111,3)).isTrue();
    }

    @Test
    @DisplayName("사용자가 3자리 수 입력하였는지, MyMathUtil적용 후 다시 확인")
    void test6(){
        BallMaker ballMaker = new BallMaker();
        
        assertThat(ballMaker.checkNum(11)).isFalse();
        assertThat(ballMaker.checkNum(1111)).isFalse();
        
        assertThat(ballMaker.checkNum(111)).isTrue();
    }
```

- checkDigitUpper와 CheckDigitLower를 checkDigit으로 합쳤다.
- 너무 작게 나누어서 결국에는 축약되었다
  - 너무 작게 나누면 합치면 그만이지만
  - 초반에 못 나누어 버리면 많은 걸 뒤집어야 할 수도 있다
- test6은 MyMathUtil 적용 후 다시 테스트 한 것이다(test2와 같은내용)
`사실 이렇게 하면 안된다(test2와 똑같잖아..그러지마)`
  
  <br/>
  
```java
    public static boolean checkDigit(int i, int digit) {
        //return checkDigitLower(i,digit)&&checkDigitUpper(i,digit);

        //두개로 나눌 필요가 없었구나..

        //refactoring - 1
        //return i/(int)Math.pow(10,digit) == 0 && i/(int)Math.pow(10,digit-1) != 0;

        //refactoring - 2
        int n = (int)Math.pow(10,digit);
        return i/n == 0 && i/(n/10) != 0;
    }
```

---

#### 🏀 마치며...

숫자 생성 시의 3가지 조건중 겨우 2가지 밖에 안되는데

이렇게 심각하게 고민하는 시간을 가졌다.

아쉬웠던 점들을 고려하여, 내가 짠 것을 "싹 다 지우고"

다시.. 다시.. 시작할 것이다.

어쩌면 잘못된 감을 잡은건데 괜히 열심히 하는 것은 아닌가 싶다.

내가 믿을 것은 오직 팀원들과의 피드백뿐이다.

<br/>

다음 포스팅은 오늘의 아쉬운 점이 개선 된 더 나은 코드이다.

※ 리팩토링은 보이는 즉시 해야한다. 미뤄서 고생했다.

※ commit을 안하여서 이 모든 과정을 기억에 의존하여 작성하다 보니 그 당시 만큼의 생동감을 표현하지 못했다.


