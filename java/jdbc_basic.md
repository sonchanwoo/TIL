> 🚀 필요한 라이브러리 : mysql-connector-java

- maven

```java
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.27</version>
</dependency>
```

<br/>

> 🚀 code - 1

```java
public enum ConnectionUtil {
    INSTANCE;
    
    private String url = "jdbc:mysql://localhost:3306/****";
    private String username = "****";
    private String password = "****";

    ConnectionUtil() {
        try {
            
            Class.forName("com.mysql.cj.jdbc.Driver");
            
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    
    public Connection getConnection() {
        Connection connection=null;
        try {
            
            connection = DriverManager.getConnection(url,username,password);
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        return connection;
    }
}
```

1. Class.forName()
  - 내부적으로 Driver객체를 생성하여, DriverManager에 등록한다.

2. DriverManager.getConnection()
  - url, username, password 정보로 DB에 접근한다.
  
<br/>

> 🚀 code - 2

```java
public enum SampleDAO {
    INSTANCE;
    
    public void select()throws Exception{
        
        String sql = "select * from tbl_board where bno > ?";
        
        Connection connection=ConnectionUtil.INSTANCE.getConnection();
        
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 4);
        
        ResultSet resultSet = preparedStatement.executeQuery();
        
        while(resultSet.next()) {
            System.out.println(resultSet.getString(1)+", "+resultSet.getString(2)+", "+resultSet.getString(3));
        }
        
    }
}
```

1. String타입으로 sql문을 준비한다.(세미콜론 없는 것 주의!)

2. connection을 받아온다.(이전 코드)

3. preparedStatement = connection.prepareStatement(sql)
  : sql문을 준비한다.
  
4. preparedStatement.setXXX(n,value)
	: (?가 있는 경우) n번째 자리의 ?에 value를 넣는다.

5. resultSet = preparedStatement.executeQuery()
	: 쿼리 실행 결과(들)을 받아온다.
    
6. resultSet.next()
	: 기본 위치는 첫 row의 직전이다. next()를 통해 다음 값으로 넘어간다.

7. resultSet.getXXX(n) : n번째 column값을 받는다.
