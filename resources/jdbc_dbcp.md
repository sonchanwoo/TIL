> ๐ ๊ธฐ์กด ์ฝ๋์ ๋ฌธ์ ์ 

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

- ํ ๋ช์ ์ฌ์ฉ์๊ฐ ์ select๋ฉ์๋๋ฅผ ์ฌ์ฉํ๋ ์๋น์ค๋ฅผ ์ด์ฉํ๋ค๊ณ  ๊ฐ์ .
  - ConnectionUtil์ getConnection์ ํตํด connection์ ์๋ก ๋ฐ๋๋ค.
  - ๋ณผ ์ผ์ด ๋๋ฌ์ผ๋ฉด connection์ closeํ๋ค(@Cleanup)  

  -> ํ ๋ช์ด ์ง์  ๊ธ๊ณ ์ ๋ค์ด๊ฐ ๋์ ๋ค๊ณ  ๋์จ๋ค.
  
- +๋์์ ์ฌ๋ฌ๋ช์ด ์ ์ํ๋ฉด?
  -> ๋ฐฑ ๋ช์ด ๋์์ ๊ธ๊ณ ์ ๋ค์ด๊ฐ ๋์ ๋ค๊ณ  ๋์จ๋ค.
  
- +์กฐํ(select)๋ฟ ์๋๋ผ ๋จ์๊ฐ์ ๋ค๋ฅธ ์ฟผ๋ฆฌ๋ ์ด์ฉํ  ๊ฒ์ด๋ค.
  -> ๋๊ตฐ๊ฐ๋ ์๊ธ, ์กฐํ, ์ด์ฒด, ๋ ์กฐํ..
  -> ๋ค๋ง "์ฝ๋ ์", ์ 4๊ฐ์ง ์๋ฌด๋ฅผ ๊ธ๊ณ ์ ๋ค์ด๊ฐ ํ ํ ๋ฒ์ ํ๋ ๊ฒ ์๋๋ผ ๊ธ๊ณ ์ ๋ค์ด๊ฐ ์๊ธ์ ํ๊ณ , ๋๊ฐ๋ค ๋ค์ ๋ค์ด๊ฐ์ ์กฐํ, ๋๊ฐ๋ค ๋ค์ ๋ค์ด๊ฐ์ ์ด์ฒด... 

- +ํ๋์ ์๋น์ค๊ฐ ๋ ๊ฐ ์ด์์ ์ฟผ๋ฆฌ๋ฌธ์ผ๋ก ์ด๋ฃจ์ด์ ธ ์๋ค๋ฉด?
  -> ..?!??!
  
<br/>

> ๐ ๋ฐ์ดํฐ๋ฒ ์ด์ค ์ปค๋ฅ์ ํ(DBCP)์ ์ฌ์ฉํ๋ ์ด์ 
- ์์ ์ด์ ๋ก ์ธํ DB ๊ณผ๋ถํ๋ฅผ ๋ง๊ธฐ ์ํด(์๋ฌ๊ฐ ๋๋ฉด ์ฌ์ ์์ ํด์ผํจ)
- ์ปค๋ฅ์์ ์ ํด์ง ๊ฐ์ ๋งํผ '๋ฏธ๋ฆฌ' ๋ง๋ค์ด ๋์ด Pool์ ํ์ฑํ๋ค.
- ๋ฏธ๋ฆฌ ๋ง๋ค์ด ๋์๊ธฐ ๋๋ฌธ์ ์์ฑ ์ ์๋น์๊ฐ์ด ๋ค์ง ์๋๋ค.
- pool์์ ๋น๋ฆฌ๊ณ , ๋ฐ๋ฉํ๊ณ .. '์ฌ์ฌ์ฉ'ํ๋ค.
- ๋น๋ฆด ์ ์์ ๊ฒฝ์ฐ '๋๊ธฐ' ์ํ๋ฅผ ์ค ์ ์๊ฒ ํ๊ธฐ ์ํด.

1. ๋ผ์ด๋ธ๋ฌ๋ฆฌ ์ถ๊ฐ

```xml
<!-- https://mvnrepository.com/artifact/com.zaxxer/HikariCP -->
<dependency>
	<groupId>com.zaxxer</groupId>
	<artifactId>HikariCP</artifactId>
	<version>5.0.0</version>
</dependency>
```

<br/>

2. ๋น ๋ฑ๋ก

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

- ๋๋ผ์ด๋ฒ ๋ก๋ฉ ๋ฐ ์ ๋ณด๋ค(url,id,pw)๋ก ์ปค๋ฅ์ ๋ฐํ์ด ํฌํจ๋๋ค.
- dataSource๋ผ๋ ์ด๋ฆ์ ์ปค๋ฅ์ ํ ์์ฑ

![](https://images.velog.io/images/sonchanwoo/post/d9794a12-91a0-4ea7-857a-4bdbee60753a/image.png)

<br/>

3. ๊ธฐ์กด ์ฝ๋์ ์์ 

- ConnectionUtil.java ์ญ์ 
  - ๋๋ผ์ด๋ฒ ๋ก๋ฉ๊ณผ, ์ ์ ํ ์ ๋ณด๋ค๋ก ์ปค๋ฅ์์ ๋ฐ๋ ๊ฒ์ HikariConfig๊ฐ ํด์ฃผ์์ผ๋ฏ๋ก
  - ๊ทธ๋ฐ ์ปค๋ฅ์๋ค์ด ๋ชจ์ฌ์๋ dataSource๊ฐ ์์ผ๋ฏ๋ก
  
<br/>

- SampleDAO ์์ 
  - ์ด๋ฏธ ๋ฑ๋ก๋ผ์๋ dataSource๋ผ๋ ๋น์ ๋จ์ผ ์์ฑ์ ๋ฌต์์  ์๋์ฃผ์์ผ๋ก
(@Component + @AllArgsConstructor + component-scan(์ฝ๋์๋ต))

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
        
        //์๋ต
        
        }catch(SQLException e) {
            e.printStackTrace();
        }        
    }
}

```

<br/>

- ํ์คํธ ์ฝ๋ ์์ 
  ๊ธฐ์กด enum์ผ๋ก ์ฑ๊ธํค ๊ตฌํ -> bean์ผ๋ก ๊ตฌํ

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