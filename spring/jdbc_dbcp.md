> 🚀 기존 코드의 문제점

```java
  public void select(){                
      String sql = "select * from tbl_board where bno > ?";   
        
      try {
      @Cleanup Connection connection=ConnectionUtil.INSTANCE.getConnection();
      @Cleanup PreparedStatement preparedStatement = connection.prepareStatement(sql);
      preparedStatement.setInt(1, 4);
      @Cleanup ResultSet resultSet = preparedStatement.executeQuery();
        
      while(resultSet.next()) {
            resultSet.getInt(1);
            resultSet.getString(2);
            resultSet.getString(3);
      }
        
      }catch(SQLException e) {
          e.printStackTrace();
      }        
  }
```

- 한 명의 사용자가 위 select메서드를 사용하는 서비스를 이용한다고 가정.
  - ConnectionUtil의 getConnection을 통해 connection을 새로 받는다.
  - 볼 일이 끝났으면 connection을 close한다(@Cleanup)  

  -> 한 명이 직접 금고에 들어가 돈을 들고 나온다.
  
- +동시에 여러명이 접속하면?
  -> 백 명이 동시에 금고에 들어가 돈을 들고 나온다.
  
- +조회(select)뿐 아니라 단시간에 다른 쿼리도 이용할 것이다.
  -> 누군가는 입금, 조회, 이체, 또 조회..
  -> 다만 "코드 상", 위 4가지 업무를 금고에 들어간 후 한 번에 하는 게 아니라 금고에 들어가 입금을 하고, 나갔다 다시 들어가서 조회, 나갔다 다시 들어가서 이체... 

- +하나의 서비스가 두 개 이상의 쿼리문으로 이루어져 있다면?
  -> ..?!??!
  
<br/>

> 🚀 데이터베이스 커넥션 풀(DBCP)을 사용하는 이유
- 위의 이유로 인한 DB 과부하를 막기 위해(에러가 나면 재접속을 해야함)
- 커넥션을 정해진 개수 만큼 '미리' 만들어 두어 Pool을 형성한다.
- 미리 만들어 두었기 때문에 생성 시 소비시간이 들지 않는다.
- pool에서 빌리고, 반납하고.. '재사용'한다.
- 빌릴 수 없을 경우 '대기' 상태를 줄 수 있게 하기 위해.

1. 라이브러리 추가

```xml
<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<dependency>
	<groupId>com.zaxxer</groupId>
	<artifactId>HikariCP</artifactId>
	<version>5.0.0</version>
</dependency>
```

<br/>

2. 빈 등록

```xml
<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
	<property name="driverClassName"
		 value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy">
        </property>
        
	<property name="jdbcUrl" value="jdbc:log4jdbc:mysql://localhost:3306/****" >
	</property>
    
	<property name="username" value="****"></property>
	<property name="password" value="****"></property>
</bean>

<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
	<constructor-arg ref="hikariConfig" />
</bean>
```

- 드라이버 로딩 및 정보들(url,id,pw)로 커넥션 반환이 포함된다.
- dataSource라는 이름의 커넥션 풀 생성

![](https://images.velog.io/images/sonchanwoo/post/d9794a12-91a0-4ea7-857a-4bdbee60753a/image.png)

<br/>

3. 기존 코드의 수정

- ConnectionUtil.java 삭제
  - 드라이버 로딩과, 적절한 정보들로 커넥션을 받는 것을 HikariConfig가 해주었으므로
  - 그런 커넥션들이 모여있는 dataSource가 있으므로
  
<br/>

- SampleDAO 수정
  - 이미 등록돼있는 dataSource라는 빈을 단일 생성자 묵시적 자동주입으로
(@Component + @AllArgsConstructor + component-scan(코드생략))

  - connection = dataSource.getConnection()

```java
@Component
@AllArgsConstructor
public class SampleDAO {
    
    private DataSource dataSource;
    
    public void select(){                
        String sql = "select * from tbl_board where bno > ?";   
        
        try {
        @Cleanup Connection connection=dataSource.getConnection();
        @Cleanup PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 4);
        @Cleanup ResultSet resultSet = preparedStatement.executeQuery();
        
        //생략
        
        }catch(SQLException e) {
            e.printStackTrace();
        }        
    }
}

```

<br/>

- 테스트 코드 수정
  기존 enum으로 싱글톤 구현 -> bean으로 구현

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/**/root-context.xml"})
public class Tests {
    
    @Setter(onMethod_ = @Autowired)
    private SampleDAO dao;
    
    @Test
    public void test1() throws Exception {
        //SampleDAO.INSTANCE.select();
        dao.select();
    }
}
```