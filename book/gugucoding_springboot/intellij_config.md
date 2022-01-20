![](https://images.velog.io/images/sonchanwoo/post/caea4888-fe0d-42be-9378-cdcd88b0ac03/20220119_151247.png)

- 빨간색 사항 확인

- 파란색(프로젝트명)은 하나만 수정해도 같이 간다.

<br/>

![](https://images.velog.io/images/sonchanwoo/post/ab1521ff-82f0-4e48-8f09-6a42c024db74/20220119_151452.png)

- spring initializr의 dependancies

<br/>

![](https://images.velog.io/images/sonchanwoo/post/26be9a6d-93b6-49e8-b34e-47f4cc10fe96/image.png)

- edit configurations의 두 항목 다 update classes and resources

<br/>

```gradle
dependancies{

    ...
    
    //test에서도 lombok을 사용하고 싶을 때
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
    
    // https://mvnrepository.com/artifact/mysql/mysql-connector-java
    // mysql 커넥터
    implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.27'
    
    //타임리프에서 dateFormat을 사용하기 위함
    implementation group: 'org.thymeleaf.extras', name: 'thymeleaf-extras-java8time'

    ...
}
```

- build.gradle의 dependancies

<br/>

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/****
spring.datasource.username=****
spring.datasource.password=****

spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true

spring.thymeleaf.cache=false
```

- resources의 application.properties파일
- hikariCP 기본적으로 사용
- 책은 둘째 공주님 mariaDB를 사용하지만, 설치문제로 mysql로 따라가기로 하였다.