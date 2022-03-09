> 출처 : 코드로 배우는 스프링 웹 프로젝트(구멍가게코딩단)

---

1. pom.xml

```xml
	<!-- https://mvnrepository.com/artifact/org.quartz-scheduler/quartz -->
	<dependency>
		<groupId>org.quartz-scheduler</groupId>
		<artifactId>quartz</artifactId>
		<version>2.3.0</version>
	</dependency>


	<!-- https://mvnrepository.com/artifact/org.quartz-scheduler/quartz-jobs -->
	<dependency>
		<groupId>org.quartz-scheduler</groupId>
		<artifactId>quartz-jobs</artifactId>
		<version>2.3.0</version>
	</dependency>
```

2. root-config

```xml
	<context:component-scan
		base-package="org.zerock.task"></context:component-scan>

	<task:annotation-driven/>
```

3. dao

```java
	@Select("select * from t_attach where path = date_format(sysdate() - interval 1 day,'%Y\\%m\\%d')")
    List<AttachVO> getOldFiles();
```

- 매일 특정 시간에 위 메서드로 불러온 것이 아닌 파일들을 삭제할 것이다.

- 어제 날짜의 모든 사진들을 가지고 온다.(이틀 전꺼는 전날에 처리 했었을 것이니)

4. org.zerock.task.FileCheckTask.java

```java
@Component
public class FileCheckTask {

    @Setter(onMethod_ = { @Autowired })
    private AttachDAO dao;

    private String getFolderYesterDay() {

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

        Calendar cal = Calendar.getInstance();

        cal.add(Calendar.DATE, -1);

        String str = sdf.format(cal.getTime());

        return str.replace("-", File.separator);
    }

	//매일 15시44분0초에
    @Scheduled(cron = "0 44 15 * * *")
    public void checkFiles() throws Exception {
        // DB에 있는 파일들을 불러온다.
        List<AttachVO> fileList = dao.getOldFiles();

        //각 컬럼의 정보들로 파일 이름을 만들어 리스트에 담는다.
        List<Path> fileListPaths = fileList.stream()
                .map(vo -> Paths.get("c:\\resources", vo.getPath(), vo.getUuid() + "_"+vo.getName()))
                .collect(Collectors.toList());

		//섬네일 파일도 리스트에 추가한다.
        fileList.stream()
        .map(vo -> Paths.get("c:\\resources", vo.getPath(), "s_" + vo.getUuid() + "_" + vo.getName()))
        .forEach(p -> fileListPaths.add(p));

        //전날 날짜의 폴더
        File targetDir = Paths.get("c:\\resources", getFolderYesterDay()).toFile();

		//지울 파일들 = 전날 날짜의 폴더의 파일들에서 리스트가 아닌 것들만 가져온다.
        File[] removeFiles = targetDir.listFiles(file -> fileListPaths.contains(file.toPath()) == false);

		//지운다.
        for (File file : removeFiles) {
            file.delete();
        }
    }
} 
```