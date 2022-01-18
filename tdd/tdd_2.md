방금 전까지, 여태 해온 숫자 야구게임 코드들을
모든 커밋 순으로 나열하여, 변천사 및 의사결정 과정을 담으려고 했었다.

하지만 한 기능 구현 단위로 커밋을 지키지 않았기 때문에 어려움이 있어
모두 삭제하였다.

아직 리팩토링 할 부분이 많이 남았지만, 지금까지의 최종본 리뷰를 진행하겠다.

<br/>

> 🚀 to do list와 Class 변화 과정
- 자세한 코드와 테스트 코드는 생략하였음

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드

->

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - 사용자의 입력이 타당한지 검사하는 check() 같은..
  
->

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - 사용자의 입력이 타당한지 검사하는 check() 같은..
    - 3자리
    - 1~9 사이
    - 중복x
    
-> 

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - 사용자의 입력이 타당한지 검사하는 check() 같은..
    - 3자리 : 입력 받은 수가 n자리 인지 검사하는 것은 BallMaker의 성격은 아니며, 마치 util같다고 느꼈다.
    - 1~9 사이
    - 중복x
    
```java
class BallMaker{
  private checkNum(){
    //내부적으로 util을 사용한다.
    MyMathUtil.checkDigit()
  }
}
```

```java
class MyMathUtil{
  public checkDigit(num,digit){
    //num이 digit 자릿수인지 검사한다.
  }
}
```
    
->

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - 사용자의 입력이 타당한지 검사하는 check() 같은..
    - ~~3자리~~ ( clear! )
    - 1~9 사이 : 세자리 수를 분해한다 -> 하나하나가 1~9인가? -> 모두 1~9인가
    - 중복x
    
```java
class MyMathUtil{
  public checkDigit();
  
  public divide();
}
```
    
```java
class BallMaker{
  private checkNum();
  
  //하나하나가 1~9인가
  private checkRangeOfOneNum();
  
  private checkRange(){
    MyMathUtil.divide();//수를 나눈다
    while(){//모두 1~9인가
      checkRangeOfOneNum();
    }
  }
}
```

->

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - 사용자의 입력이 타당한지 검사하는 check() 같은..
    - ~~3자리~~
    - ~~1~9 사이~~
    - 중복x : 숫자들을 분해 -> 숫자 하나가 (임시)리스트에 없는지 검사한다 -> 없으면 넣는다 -> 중복없이 무사히 다 돌거나, 도중에 중복이 생기면 반복문을 탈출한다.
    
```java
class MyMathUtil{
  public checkDigit();
  
  public divide();
}
```
    
```java
class BallMaker{
  private checkNum();
  
  private checkRangeOfOneNum();  
  private checkRange();
  
  //num이 list에 있는지 검사한다.
  private checkDuplicatedOfOneNum(num,list);
  private checkDuplicated(num){
    nums = MyMathUtil.divide();
    list_tmp//임시 저장소
    
    while(){
      checkDuplicatedOfOneNum(num,list_tmp);//중복이면 다음 루프를 돌지 않게 설계
      list_tmp.add(num)//중복이 안되면 list에 추가한다.
    }
  }
}
```

->

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - 사용자의 입력이 타당한지 검사하는 check() 같은..
 
```java
class MyMathUtil{
  public checkDigit();
  
  public divide();
}
```
    
```java
class BallMaker{
  private checkNum();
  
  private checkRangeOfOneNum();  
  private checkRange();
  
  private checkDuplicatedOfOneNum();
  private checkDuplicated();
  
  private check(){
    checkNum && !checkDuplicated && checkRange
  }
}
```

-> 

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - ~~사용자의 입력이 타당한지 검사하는 check() 같은..~~
  - create고 뭐고 Ball,Balls 부터 만들어야겠는데?
  
```java
class Ball{
  int num
  
  //생성자
  Ball(num){}
}
```

```java
class Balls{
  List<Ball> list;
  
  //생성자
  Balls(List<Ball> list){
    this.list = list
  }
}
```

->

- 사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드
  - ~~사용자의 입력이 타당한지 검사하는 check() 같은..~~
  - ~~create고 뭐고 Ball,Balls 부터 만들어야겠는데?~~
  - createBall -> createBalls(지금 보니 createBall은 괜히 만들었다고 생각한다.)
  
```java
class BallMaker{
  private checkNum();
  
  private checkRangeOfOneNum();  
  private checkRange();
  
  private checkDuplicatedOfOneNum();
  private checkDuplicated();
  
  private check();
  
  private createBall(num){
    new Ball(num)
  }
  
  public createUserBalls(num){
    if(!check(num)) return null;
    
    numbers = MyMathUtil.divide();
    List<Ball> balls
    
    while -> balls.add(createBall(number))
    
    return new Balls(balls);
  }
}
```

->

- ~~사용자의 입력으로 Balls를 만드는 BallMaker 클래스의 create() 식의 메서드~~

- createComputerBalls()
  - 1~9 사이, 자릿수는 지키기 쉬움, 중복만 유의!
  - createComputerBall : 임시리스트에서 없는 랜덤값 나올때까지 반복 -> createComputerBalls : 임시리스트 늘려가며 Balls 생성
  
```java
class BallMaker{
  private checkNum();
  
  private checkRangeOfOneNum();  
  private checkRange();
  
  private checkDuplicatedOfOneNum();
  private checkDuplicated();
  
  private check();
  
  private createBall();  
  public createUserBalls();
  
  private createComputerBall(list){//임시리스트
  
    //아까 만든 것 이용
    while(checkDuplicatedOfOneNum(num, list)){
      //중복 안되는 랜덤값 나올때까지 계속..
      num = (int) (Math.random() * MAX_NUM) + MIN_NUM;
    }
    
    new Ball(num)
  }
  
  public createComputerBalls(){
    while(){
      ball = createComputerBall(list);
      balls.add(ball);
      list.add(ball.getNum());//임시리스트 늘리기
    }
    new Balls(balls)
  }
}
```

-> 사용자와 컴퓨터의 규칙에 맞는 숫자 생성까지 완료, 이하 클래스들 간단 요약.

```java
class BallMaker{//private 생략  
  public createUserBalls();  
  public createComputerBalls();
}

class Ball{}
class Balls{}

class MyMathUtil{
  public checkDigit();
  public divide();
}
```

<br/>

- 두 Balls를 비교하는 BallMacther 클래스
  - 1볼2스트라이크, 3스트라이크, 낫띵 등 전체결과를 의미하는 어떤 info 성질의 클래스
    - Ballstate라는 상태가 먼저필요
    
```java
public enum BallState {
    BALL, STRIKE, NOTHING;
}
```

->

- 두 Balls를 비교하는 BallMacther 클래스
  - 1볼2스트라이크, 3스트라이크, 낫띵 등 전체결과를 의미하는 어떤 info 성질의 클래스
    - ~~Ballstate라는 상태가 먼저필요~~
    
```java
public class BallStatesInfo {
    private int ballCnt;
    private int strikeCnt;

    //명세 반환
    @Override
    public String toString(){
        StringBuffer message = new StringBuffer("");

        int ballCnt = this.ballCnt;
        int strikeCnt = this.strikeCnt;

        if(ballCnt==0&&strikeCnt==0)
            return "Nothing";

        if(ballCnt>0)
            message.append(ballCnt+"볼 ");
        if(strikeCnt>0)
            message.append(strikeCnt+"스트라이크 ");

        return message.toString();
    }
}
```

->

- 두 Balls를 비교하는 BallMacther 클래스
  - ~~1볼2스트라이크, 3스트라이크, 낫띵 등 전체결과를 의미하는 어떤 info 성질의 클래스~~
  - userBall하나와 computerBall하나를 비교하여
  BallStatesinfo에 ballCnt, strikeCnt를 up하는 논리
    - ball과 strike를 비교하기 위해 위치 성분이 필요해짐
    -> Balls의 인덱스를 주어도 되지만, Ball 자체적으로 가지고 있는게 의미상 자연스럽다고 판단!
    
```java
//Ball 클래스의 수정
class Ball {//
    int num;
    int position;

    Ball(int num);

    // num과 position이 둘다 같아야만 같은걸로 수정(스트라이크)
    equals();
}

//Balls 클래스의 수정
class Balls{
  List<Ball> list;
 
  Balls(List<Ball> list){
    this.list = list;
    setPositions();//리스트 내부의 볼들에게 list인덱스 값을 set!
  }
}
```

->

- 두 Balls를 비교하는 BallMacther 클래스
  - ~~1볼2스트라이크, 3스트라이크, 낫띵 등 전체결과를 의미하는 어떤 info 성질의 클래스~~
  - userBall하나와 computerBall하나를 비교하여
  BallStatesinfo에 ballCnt, strikeCnt를 up하는 논리
    - ~~ball과 strike를 비교하기 위해 위치 성분이 필요해짐
    -> Balls의 인덱스를 주어도 되지만, Ball 자체적으로 가지고 있는게 의미상 자연스럽다고 판단!~~
    
```java
public class BallMatcher {

    private void countBallState(Ball userBall,Ball computerBall, BallStatesInfo ballStatesInfo){
        if(userBall.equals(computerBall)) {
            ballStatesInfo.countUp(BallState.STRIKE);
            return;
        }
        if(userBall.getNum() == computerBall.getNum()){
            ballStatesInfo.countUp(BallState.BALL);
            return;
        }
    }
}

public class BallStatesInfo {
    private int ballCnt;
    private int strikeCnt;

    //구종에 따라 올린다
    public void countUp(BallState ballState){
        if(ballState.equals(BallState.BALL))
            this.ballCnt++;
        if(ballState.equals(BallState.STRIKE))
            this.strikeCnt++;
    }
}
```

->

- 두 Balls를 비교하는 BallMacther 클래스
  - ~~userBall하나와 computerBall하나를 비교하여
  BallStatesinfo에 ballCnt, strikeCnt를 up하는 논리~~
  - -> userBall하나와 computerBall 전체
  - -> userBall전체와 computerBall 전체
  -> 이 모든걸 이용한 BallStatesInfo를 반환하는 getBallStatesInfo( )메서드
  
```java
class BallMaker{
  public createUserBalls();  
  public createComputerBalls();
}

class Ball{}
class Balls{}

class MyMathUtil{
  public checkDigit();
  public divide();
}

enum BallState{}

class BallStatesInfo{
  public countUp();
}

class BallMatcher{
  public getBallStatesInfo(){
    //내부적으로 ballStatesInfo를 생성
    //루프를 돌며 ballStatesInfo의 countUp()하며
    //완성된 Info반환.
  }
}
```

-> 

- ~~BallMatcher~~

- 이제 남은 건 게임논리 -> Scanner와 main함수 때문에 test가 어려워 그냥 짰다.

<br/>

`전체 소스 코드`

https://github.com/sonchanwoo/java-baseball-playground.git


<br/>

> 🚀 후기

- 나름 노하우가 생긴 것 같다.

1. 리팩토링을 미루면 대부분 고생한다.

2. 물론, 미루어도 되는 리팩토링 대상이 있기는 하다.
(아직까진 말로 설명하기는 어려우니, 나중에 정립해보아야겠다.)

3. 테스트 코드는 중요하다.
  (1) A클래스의 테스트 코드는, A를 사용하는 외부의 로직을 짤 때 힌트가 된다.
  (2) 수정을 해야할 때, 흔들리지 않게 해주는 중요한 자원이다.
  (이것도 말로 설명은 어려우니 2번과 함께 예시를 들어야겠다.)

<br/>

- 다음에 해야할 것

1. 지속적인 리팩토링

2. 피드백 반영

3. 강의 다시 보기

4. 우테코 프리코스 제출자 참고

5. 추가적인 요구사항 적용(15라인 미만 등)






