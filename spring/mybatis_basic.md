> 🚀 라이브러리 추가

```
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>${org.springframework-version}</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-tx -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${org.springframework-version}</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.5.7</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>1.3.2</version>
</dependency>

```

<br/>

> 🚀 SQLSession, SQLSessionFactory
1. SQLSessionFactorys는 SQLSession객체를 만들어 낸다.
2. SQLSession으로 connection 생성, SQL질의, 결과 리턴 등을 할 수 있다.

1. bean 등록

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

![](https://images.velog.io/images/sonchanwoo/post/d8235b13-b330-4a57-91d6-18d1a02e4caa/image.png)

- dataSource를 참조함을 알 수 있다.

<br/>

2. 코드 수정

- 팩토리에서 세션을 만들어 내어 커넥션을 받아온다.
- 기존 dataSource를 대체한다.

```java
@Component
@AllArgsConstructor
public class SampleDAO {

    //private DataSource dataSource;
    private SqlSessionFactory sqlSessionFactory;

    public void select(){                
        String sql = "select * from tbl_board where bno > ?";   

        try {
            @Cleanup SqlSession session = sqlSessionFactory.openSession();
            @Cleanup Connection connection=session.getConnection();
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
}
```

- session으로 커넥션을 얻어 오는 것 외에 질의도 할 수 있지만, 다음에 알아보도록 하고, mybatis의 매퍼를 사용하여 보겠다.

<br/>

> 🚀 Mapper - 1 (annotation)

1. DTO작성

```java
@Data
public class SampleDTO {
    private int bno;
    private String title;
}
```

2. SampleDAO 수정

```java
public interface SampleDAO {
    @Select("select * from tbl_board where bno > 4")
    List<SampleDTO> select();
}
```

3. root-context.xml 수정

```
<mybatis-spring:scan base-package="com.company.dao" />
<!-- 
<context:component-scan	base-package="com.company.dao">
</context:component-scan>
 -->
```

4. 테스트 코드(이전과 같음)

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/**/root-context.xml"})
public class Tests {
    
    @Setter(onMethod_ = @Autowired)
    private SampleDAO dao;
    
    @Test
    public void test1() throws Exception {
        dao.select();
    }
}
```

<br/>

5. 결과 - 성공

![](https://images.velog.io/images/sonchanwoo/post/3230587a-f02c-4da2-96f1-1d0225beada4/image.png)

<br/>

?????????????????

<br/>

- 어노테이션 처리를 하였으니 statement 준비(prepare), 집행(execute), 반환(resultSet)까지는 이해하겠다.
  - 커넥션은 어떻게 구했지?

- DTO 필드와 DB컬럼의 이름이 같아서 알아서 들어간 것일까, 웹에서의 EL이 getter 프로퍼티를 이용하는 것처럼, setter의 프로퍼티와 같아서 들어간 것일까?  
  - 이름이 다르면 어쩌지?

- select * from 으로 컬럼 6개를 구해왔지만, DTO에 있는 것들로만 저장됨을 확인할 수 있다.

- 알아서 List로 리턴된 것 인상적이다.
  
- 전체적으로 conn, stmt, rs 및 return 과정이 생략되었다고 볼 수 있다.
  
- context:component-scan + @Component 이 아닌
  
  - mybatis-spring:scan base-package 방식으로 bean이 생겼다.
  (SampleDAO는 interface인데 적당한 객체가(bean) 생겼다.)
  
![](https://images.velog.io/images/sonchanwoo/post/66cdb802-de85-456f-954a-36976539c56b/image.png)

<br/>

---

Q. 커넥션은 어떻게 구했지?
  -> 왠지 sqlFactory를 mybatis에서 알아서 쓸 것 같다.(아까 예제 실습하면서 이럴거면 왜썼지 했다..)
  -> 없애고 에러나면 내 말이 맞다.
  
![](https://images.velog.io/images/sonchanwoo/post/5d6eb52d-7dd8-4561-b27e-c7a6792be6d4/image.png)

- 잠깐 없애고 test돌려보았다

![](https://images.velog.io/images/sonchanwoo/post/643d93fd-2a32-4af2-a44f-7e8e73e57987/image.png)

<br/>

Q. 필드명과 컬럼명

```java
@Data
public class SampleDTO {
    
    private int bno;
    private String title2;
    
    public void setTitle(String title2) {
        this.title2 = title2;
    }
}
```

- setter 프로퍼티방식이 맞았다.
- 이름이 안맞으면 에러는 안나고 쿨하게 안 받아버린다.

<br/>

>  🚀 Mapper - 2 (xml)
- pstmt.setXXX() 처럼 치환하고 싶다!!
- 그 밖에 복잡한 sql을 사용하기 위함
- 물론, 어노테이션, xml 두 방식을 혼용할 수 있음

1. SampleDAO 수정 (어노테이션 삭제)

```java
public interface SampleDAO {
    List<SampleDTO> select();
}
```

2. xml 파일 작성

![](https://images.velog.io/images/sonchanwoo/post/c4ebf9fe-6b9b-4e76-babe-35d458cd9476/image.png)

- resources에 작성
- 경로를 같게 해주면 가독성 👌👌
- 폴더는 하나씩 만들 것

<br/>

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
 
<mapper namespace="com.company.dao.SampleDAO">
        
        <select id="select" resultType="com.company.dto.SampleDTO" >
            select * from tbl_board
        </select>

</mapper>
```

- namespace와(인터페이스 경로) id가(메서드명) 관건
- 쿼리문 세미콜론 주의
- resultType이 DTO하나 임에도 외부에서 List로 받으면 받아지는 게 인상적

<br/>

3. test (변경없음) - 성공

<br/>

---

- Alias 적용

1. sqlFactory property 추가

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource"></property>
  <property name="configLocation" value="classpath:/mybatis-config.xml"></property>
</bean>
```

2. resources에 mybatis-config.xml추가

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration 
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    
<configuration>
  <typeAliases>
    
    <typeAlias type="com.company.dto.SampleDTO" alias="SampleDTO" />
    
  </typeAliases>
</configuration>
```

3. mapper.xml의 resultType 변경

```xml
        <select id="select" resultType="SampleDTO" >
            select * from tbl_board
        </select>
```

<br/>

---

- 치환 적용

1. xml수정

```xml
	<select id="select" resultType="SampleDTO">
        <![CDATA[
            select bno from tbl_board where bno > #{bno}
            ]]>
        <!-- <, > 등을 태그로 인식하기 때문에 <![CDATA[ ]]>를 사용 -->
	</select>
```

2. DAO 인터페이스 수정 및 추가

```java
public interface SampleDAO {
    List<SampleDTO> select(int bno);
    //List<SampleDTO> select(SampleDTO dto);
}
```

3. tests 수정

```java
    @Test
    public void test1() throws Exception {
        dao.select(4);
        
        /*
        SampleDTO dto = new SampleDTO();
        dto.setBno(4);
        dao.select(dto);
        */
    }
```

- 주석 안, 주석 밖 둘 다 가능하다.

- 객체를 넣었을 경우, getter 프로퍼티로 동작한다.(실험완료)

- 기본 자료형이 들어가는 경우 조금 더 공부해보아야 할 것 같다.
  
  
  