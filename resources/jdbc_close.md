> ๐ Autoclosable

1. connection, (prepared)Statement, Scanner, ๊ธฐํ buffer์ข๋ฅ ๋ฑ์ ๊ฐ์ฒด์๋ close()๋ผ๋ ๋ฉ์๋๊ฐ ์๋ค.

2. ์ด๋ close()๊ฐ Autoclosable์ด๋ผ๋ ์ธํฐํ์ด์ค์ ์ถ์ ๋ฉ์๋์ด๊ธฐ ๋๋ฌธ์ด๋ค.

3. ์ด์ ๊ฐ์ ๊ฐ์ฒด๋ค์ ํ  ์ผ์ ๋ค ํ๋ฉด ๊ผญ close()ํด์ฃผ์ด์ผ ํ๋ค.

<br/>

>๐ ์ง๋ ๋ฒ ์ฝ๋์ ์์ 

1. ๊ธฐ์กด ์ฝ๋

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

2. close( ) ํด์ค ๋ฒ์ 

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
        //์๋ต
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
- close()๋ ์์ฑ ์ญ์์ผ๋ก ํ๋ค.
- ๋ชจ๋  ๊ฐ์ฒด๊ฐ ์ ๋ถ close๋์ง ์์ ์๋ ์์ผ๋ finally๋ธ๋ญ์ ๊ฒ์ฌ๋์ ์์ฑํ๋ค. (NullPointerException ๋ฐฉ์ง๋ฅผ ์ํด ์กฐ๊ฑด๋ฌธ์ ์ถ๊ฐํ๋ค.)

<br/>

3.  try - with - resource ํ์ฉํ ๋ฒ์ 

```java
public void select(){                
        try(
                Connection connection=ConnectionUtil.INSTANCE.getConnection();
                PreparedStatement preparedStatement = make(connection);
                ResultSet resultSet = preparedStatement.executeQuery();
                ) {        

        //์๋ต
        
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

- try - with - resource์ ( )์๋ AutoClosable ๊ตฌํ๊ฐ์ฒด๋ค์ด ์ฌ ์ ์๋ค.
- ์ ์ธ๊ณผ ์ด๊ธฐํ๋ง ๊ฐ๋ฅํ๋ฏ๋ก, conn.prepareStatement()๊ณผ pstmt.setXXX()๋ฅผ ์ํ์ฌ ๋ฐ๋ก ๋ฉ์๋๋ฅผ ๋ถ๋ฆฌํด์ฃผ์ด์ผ ํ๋ค.

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
        
        //์๋ต
        
        }catch(SQLException e) {
            e.printStackTrace();
        }        
    }
```

- ๊ฐ์ฅ ์ข์ ์ ์๋ ๋ฐฉ๋ฒ์ด๋ค.