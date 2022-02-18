> 🚀 lamda

- 람다는 익명함수이다.

- 다음은 이렇게 표현될 수 있다.

```java
	int add(int a,int b) {
        return a+b;
    }
```

```java
	//이름과 반환타입을 생략한다.
    (int a, int b) -> {return a+b;}
    //추론이 가능한 경우에는 매개변수의 타입을 생략할 수 있다.
    //(거의 대부분의 경우 생략가능하다.)
    (a, b) -> {return a+b;}
    //body가 한줄인 경우에는 {} 및 ;을 생략할 수 있다.
    //매개변수가 하나일 때는 ()도 생략가능하다.
    (a, b) -> a+b
```

- 사용할 땐 이렇게!

```java
	BiFunction<Integer,Integer,Integer> bf = (a,b) -> a+b;       
    System.out.println(bf.apply(4,5));
    
    //2개의 매개변수와 반환타입이다.(매개변수가 2개이므로 Bi~이다.)
    //단일 매개변수의 경우 Fucntion을 사용하면 된다.
    //apply()에 인자를 주어 excute한다.
```

```java
	//대부분의 경우 위 방식보단 아래처럼 녹아들게 사용한다.
    IntStream.range(0, 1).forEach(i -> System.out.print(i));
```

- 메서드 참조

```java
	public static void main(String[] args) {
        Function<String, String> f2 = (str) -> Main2.print(str);
        
        Function<String, String> f = Main2::print;
        //static메서드 사용할 때 처럼 클래스::메서드 또는 객체::메서드로 간다.
        //하나의 메서드만 호출하는 람다식은 위와 같이 메서드 참조가 가능하다.
    }

    public static String print(String str) {
        System.out.println(str);
        return null;
    }
```

```java
//이렇게 사용할 수 있다는 말을 하고 싶었던 것이다.
IntStream.range(0, 1).forEach(System.out::print);
```