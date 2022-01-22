
- 브라우저 설정

![](https://images.velog.io/images/sonchanwoo/post/9a7dbf3a-0750-4341-b587-ae84798b2f24/20211230_151906.png)

<br/>

- 프로젝트 생성

![](https://images.velog.io/images/sonchanwoo/post/b8452e60-cbf7-4ce4-a5cd-2a408e96242e/20211230_150739.png)

![](https://images.velog.io/images/sonchanwoo/post/3543c56c-6ec7-40ee-a606-abfb31baa7b8/20211230_150857.png)

<br/>

- 패키지 지정

![](https://images.velog.io/images/sonchanwoo/post/8c734fe2-679f-4af9-9039-d0b766b1511c/20211230_151022.png)

<br/>

- UTF-8 설정
  - workspace
  - jsp/css/html
  
<br/>

- pom.xml 설정( alt+F5[update project] 습관화 )

![](https://images.velog.io/images/sonchanwoo/post/8e67ccaf-169e-4778-8f9d-6a5fce73d06b/20211230_151405.png)

`java 11 버전`

`스프링 5.0.7 버전`

![](https://images.velog.io/images/sonchanwoo/post/30d0a569-a697-4cd6-9635-0e049d4ab0cd/20211230_151506.png)

`메이븐 컴파일러 플러그인 java-version으로`

![](https://images.velog.io/images/sonchanwoo/post/6ab3ebff-78b3-4afe-97c1-9613b4e167c5/20220111_145834.png)

`junit 4.12버전`

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
		</dependency>
		
		<!-- 
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.15</version>
			<exclusions>
				<exclusion>
					<groupId>javax.mail</groupId>
					<artifactId>mail</artifactId>
				</exclusion>
				<exclusion>
					<groupId>javax.jms</groupId>
					<artifactId>jms</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jdmk</groupId>
					<artifactId>jmxtools</artifactId>
				</exclusion>
				<exclusion>
					<groupId>com.sun.jmx</groupId>
					<artifactId>jmxri</artifactId>
				</exclusion>
			</exclusions>
			<scope>runtime</scope>
		</dependency> -->
```

`기존 log4j 삭제 또는 주석처리 -> 새 것 작성`

```xml
		<!-- Servlet -->
		<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
		</dependency>
		<!-- <dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency> -->
```
`서블릿 버전 변경`

<br/>

- 프로젝트 페이셋

![](https://images.velog.io/images/sonchanwoo/post/485241eb-d134-44a8-b5d9-696f6915c9dc/20211230_151716.png)

<br/>

- tomcat 실행시킨 후 contextRoot 및 helloWorld체킹

![](https://images.velog.io/images/sonchanwoo/post/112fb53b-e06e-4ff9-8253-bcee7366aa95/20220111_143211.png)

`톰켓 실행 시 꼭 띄울 프로젝트만 오른쪽으로`

![](https://images.velog.io/images/sonchanwoo/post/ec6ae953-1a28-4231-ac1a-10c67c31d2b5/20220111_143241.png)

`1차 확인`

![](https://images.velog.io/images/sonchanwoo/post/4bb73c98-29f4-4cb5-820b-9cade517f7f2/20220111_144804.png)

`server - server.xml의 context path 변경`

`(또는 tomcat 더블클릭 -> module의 context-root변경)`

![](https://images.velog.io/images/sonchanwoo/post/ecc79291-ed51-4780-96c8-10b6641305b1/20220111_144819.png)

`확인`