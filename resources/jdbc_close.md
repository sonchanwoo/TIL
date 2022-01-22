> 🚀 Autoclosable

1. connection, (prepared)Statement, Scanner, 기타 buffer종류 등의 객체에는 close()라는 메서드가 있다.

2. 이는 close()가 Autoclosable이라는 인터페이스의 추상 메서드이기 때문이다.

3. 이와 같은 객체들은 할 일을 다 하면 꼭 close()해주어야 한다.

<br/>

>🚀 지난 번 코드의 수정

1. 기존 코드

```java
public void select()throws Exception{
        
        String sql = "select * from tbl_board where bno > ?";
        
        Connection connection=ConnectionUtil.INSTANCE.getConnection();
        
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 4);
        
        ResultSet resultSet = preparedStatement.executeQuery();
        
    }
```

<br/>

2. close( ) 해준 버전

```java
public void select(){
        
        String sql = "select * from tbl_board where bno > ?";
        
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        
        try {
        connection=ConnectionUtil.INSTANCE.getConnection();
        preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 4);
        
        resultSet = preparedStatement.executeQuery();
        
        //...
        //생략
        //...
        
        resultSet.close();
        preparedStatement.close();
        connection.close();
        
        }catch(SQLException e) {
            e.printStackTrace();
        }finally {
            if(resultSet!=null)try {resultSet.close();} catch (SQLException e) {}
            if(preparedStatement!=null)try {preparedStatement.close();} catch (SQLException e) {}
            if(connection!=null)try {connection.close();} catch (SQLException e) {}
        }
    }
```
- close()는 생성 역순으로 한다.
- 모든 객체가 전부 close되지 않을 수도 있으니 finally블럭에 검사란을 작성한다. (NullPointerException 방지를 위해 조건문을 추가한다.)

<br/>

3.  try - with - resource 활용한 버전

```java
public void select(){                
        try(
                Connection connection=ConnectionUtil.INSTANCE.getConnection();
                PreparedStatement preparedStatement = make(connection);
                ResultSet resultSet = preparedStatement.executeQuery();
                ) {        

        //생략
        
        }catch(SQLException e) {
            e.printStackTrace();
        }
    }
    
    private PreparedStatement make(Connection connection) throws SQLException {
        String sql = "select * from tbl_board where bno > ?";   
        
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 4);
        
        return preparedStatement;        
    }
```

- try - with - resource의 ( )에는 AutoClosable 구현객체들이 올 수 있다.
- 선언과 초기화만 가능하므로, conn.prepareStatement()과 pstmt.setXXX()를 위하여 따로 메서드를 분리해주어야 한다.

<br/>

4. @Cleanup

```java
public void select(){                
        String sql = "select * from tbl_board where bno > ?";   
        
        try {
        @Cleanup Connection connection=ConnectionUtil.INSTANCE.getConnection();
        @Cleanup PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 4);
        @Cleanup ResultSet resultSet = preparedStatement.executeQuery();
        
        //생략
        
        }catch(SQLException e) {
            e.printStackTrace();
        }        
    }
```

- 가장 좋을 수 있는 방법이다.