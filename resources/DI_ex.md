`출처 : 코드로 배우는 스프링 웹 프로젝트 (구멍가게코딩단)`

> 🚀 new Object( ) 처럼 직접적으로 객체를 생성하지 않더라도
스프링은 어노테이션 등의 방법으로 객체를 생성 및 관리할 수 있는
'컨테이너'기능을 가지고 있다.
  - 스프링에서 관리되는 객체를 Bean이라고 한다.
  - root-context.xml은 Bean 설정 파일이다.

1. root-context.xml의 namespace

![](https://images.velog.io/images/sonchanwoo/post/3cfb47d8-abc2-4706-854d-609f59ef9cfd/20220111_160541.png)

- 아래 context태그를 사용하기 위함

<br/>

2. root-context.xml의 source

```xml
<context:component-scan base-package="org.zerock.sample">
</context:component-scan>
```

- 아래 Chef, Restaurant클래스들이 담겨 있는 패키지를 스캔한다.
- 스캔된 패키지의 클래스들 중에서, @Component가 있는 클래스의
객체가 생성이 된다.

<br/>

3. root-context.xml의 beans graph

![](https://images.velog.io/images/sonchanwoo/post/173b4edc-73b3-4b06-abb4-56648de33432/20220111_160647.png)

- 스캔된 패키지의 클래스들 중 @Component 어노테이션이 있는
두 클래스의 객체들(beans)

<br/>

> 🚀 @Setter와 @Autowired를 통한 의존성 주입 테스트

```java
@Component
@Data
public class Chef {
}
```

```java
@Component
@Data
public class Restaurant {

    @Setter(onMethod_ = @Autowired)
    private Chef chef;
    /*
    private Chef chef;
    
    @Autowired
    public void setChef(Chef chef){
    
    }*/
}
```

- 위의 코드를 한 번 풀어서 주석처리 해놓았다.
- @Autowired는 자신이 특성 객체에 의존적이므로, 해당하는 빈을 주입해달라는 표시이다.
bean들 중(beans graph참고) 적당한 것을 찾은 후 주입한다.

<br/>

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
public class SampleTests {

    @Setter(onMethod_ = {@Autowired})
    private Restaurant restaurant;
    
    @Test
    public void test() {
        assertNotNull(restaurant);
        assertNotNull(restaurant.getChef());
    }
}
```

- @RunWith(SpringJUnit4ClassRunner.class)
: 테스트 코드가 스프링을 실행할 것이라는 표시
- @ContextConfiguration : root-context.xml을 참고하여 스프링 빈 등록

1. 스프링 프레임워크 시작되면 스프링이 사용하는 메모리영역(컨텍스트)을 만든다.
2. root-context의 <context:component-scan>를 보고 해당하는 패키지를 스캔한다.
3. @Component이 존재하는 Chef클래스의 인스턴스(A)를 생성한다.
4. @Component이 존재하는 Restaurant클래스의 객체(B) 생성 시 A가 주입된다.
4. restaurant에 B가 주입된다.

<br/>

> 🚀 단일 생성자의 묵시적 자동 주입(스프링 4.3 버전 이후)
- 방금 전의 방법은 setter 주입
- 이제 볼 방법은 생성자 주입

- 기존 Restaurant 클래스

```java
@Component
@Data
public class Restaurant {

    @Setter(onMethod_ = @Autowired)
    private Chef chef;
}
```

- 변경

```java
@Component
@Data
@AllArgsConstructor
public class Restaurant {
    private Chef chef;
}

@Component
@Data
public class Restaurant {
    private Chef chef;
   	
    //@Autowired
    public Restaurant(Chef chef){
    	this.chef = chef
    }
}

@Component
@Data
@RequiredArgsConstructor
public class Restaurant {
    @NonNull private Chef chef;
}
```

1. 기존에는 생성자에서 @Autowired 등의 어노테이션을 추가했어야 하지만, 4.3이후에는 단일인 경우 묵시적으로 주입이 가능하다.
(생성자 2개 이상인 경우 어노테이션)

2. 위 3개의 코드는 동일하게 작동한다.