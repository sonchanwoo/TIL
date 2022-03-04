1. jsp

```jsp
	<!-- 프로필사진 -->
	<img src=""/><br/>

    <!-- 업로드바 -->
	<input type='file' name='uploadFile'><br/>

	<button id='uploadBtn'>Upload</button>
	<button id='removeBtn'>Remove</button>
	
	<br/><br/>
	
	<!-- 프로필(닉네임과 한줄소개) -->
	<input value="${member.name}" /><br/>
	<input value="${member.intro}" /><br/>

	<!-- 제이쿼리 라이브러리 -->
	<script src="https://code.jquery.com/jquery-3.3.1.min.js"
		integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8="
		crossorigin="anonymous"></script>
		
    <script type="text/javascript">
		$(document).ready(function() {
			var fileCallPath = '';
			
            //정규표현식
			var regex = new RegExp("(.*?)\.(jpg|png)$");
			
			//본 화면 도착하자마자 해당 멤버에게 저장된 프로필사진 출력
			$.getJSON("/getA?member_id=" + '${member.member_id}', function(result) {
				show(result);
			});
			
			function check(fileName) {
				//이름이 정규표현식 안에 있는지 = png / jpg인지!
				if (regex.test(fileName)) {					
					return true;
				}
				return false;
			}
			
			function show(result){
				//attachVO를 받아와서 풀-파일경로를 조합하여 img태그에 삽입
				fileCallPath = encodeURIComponent(result.path+"/s_"+result.uuid+"_"+result.name);
				document.querySelector("img").src = '/display?fileName='+fileCallPath;
			}

			$("#uploadBtn").on("click", () => {//업로드버튼 클릭 시
				var formData = new FormData();//가상의 form태그 만들어서

				var inputFile = $("input[name='uploadFile']");//파일input태그를 가져와
				var file = inputFile[0].files[0];//파일을 뽑은 후
					
				if(!check(file.name)){//이미지검사를 하고
					return;//이미지가 아니면 안업로드
				}

				formData.append("uploadFile", file);//가상의 form태그에 삽입하고
				formData.append("member_id",'${member.member_id}');//회원id도 삽입하여

				$.ajax({//호출
					url : '/upload',
					processData : false,
					contentType : false,
					data : formData,
					type : 'POST',
					dataType: 'json',
					success : function(result) {//테이블에 삽입될 vo를 도중반환한다.(쓰려고)
						show(result);
					},
				}); //$.ajax

			});
			
			$("#removeBtn").on("click",() => {// 지우기버튼을 클릭하면 컨트롤러 내부적으로 서버의 파일과, db데이터도 지운다.
				 $.ajax({
					    url: '/delete',
					    data: {fileName: fileCallPath, member_id: '${member.member_id}'},
					    dataType:'text',
					    type: 'POST',
					      success: function(result){
					         alert(result);
					       }
					  }); //$.ajax
			});
		});
	</script>
```

<br/>

2. controller

```java
@Controller
@AllArgsConstructor
@Log4j
public class PostController {
    private MemberDAO memberDAO;
    private AttachDAO attachDAO;

    @GetMapping("/info")
    public void info(Long member_id, Model model) {
        // id 값으로 멤버 반환해서 개인 설정페이지로 이동
        model.addAttribute("member", memberDAO.get(member_id));
    }

    @PostMapping("/upload")
    @ResponseBody
    public ResponseEntity<AttachVO> uploadAjaxPost(MultipartFile uploadFile, Long member_id) {
        // 회원 프로필사진 파일과 어느 회원인지 정보를 받는다.

        AttachVO vo = new AttachVO();
        String uploadFolder = "C:\\resources";

        // 폴더
        String uploadFolderPath = getFolder();// 폴더경로문자 : 2020\01\01
        File uploadPath = new File(uploadFolder, uploadFolderPath);// c:\resources\2020\01\01

        if (!uploadPath.exists()) {// 폴더가 없으면
            uploadPath.mkdirs();// 만들어
        }

        String uploadFileName = uploadFile.getOriginalFilename();

        // 쌩 파일이름만 올수도있고 폴더경로가 포함될 수도 있어서 마지막\의 이후만 빼낸다(폴더 경로가 포함된 경우에 작용)
        uploadFileName = uploadFileName.substring(uploadFileName.lastIndexOf("\\") + 1);

        UUID uuid = UUID.randomUUID();
        vo.setName(uploadFileName);// heart.jpg
        uploadFileName = uuid.toString() + "_" + uploadFileName;// heart.jpg -> uuid_heart.jpg

        File saveFile = new File(uploadPath, uploadFileName);
        // c:\resources\2020\01\01에 uuid_heart.jpg 빈파일 생성

        try {
            uploadFile.transferTo(saveFile);// 빈 파일을 파라미터로 넘어온 파일로 채우기

            // 잔여 정보들
            vo.setMember_id(member_id);
            vo.setUuid(uuid.toString());
            vo.setPath(uploadFolderPath);

            // 이미지 타입이면 썸네일 생성
            if (checkImageType(saveFile)) {
                FileOutputStream thumbnail = new FileOutputStream(new File(uploadPath, "s_" + uploadFileName));
                Thumbnailator.createThumbnail(uploadFile.getInputStream(), thumbnail, 100, 100);
                thumbnail.close();
            }
        } catch (Exception e) {
            log.error(e.getMessage());
        }
        attachDAO.insert(vo);// DB에 삽입

        return new ResponseEntity<>(vo, HttpStatus.OK);
    }

    @GetMapping("/getA") // attachVO를 불러온다.
    @ResponseBody
    public ResponseEntity<AttachVO> getA(Long member_id) {
        return new ResponseEntity<>(attachDAO.get(member_id), HttpStatus.OK);
    }

    @GetMapping("/display") // filename(사진주소)를 받아서 사진을 가져온다.
    @ResponseBody
    public ResponseEntity<byte[]> getFile(String fileName) {

        File file = new File("c:\\resources\\" + fileName);

        ResponseEntity<byte[]> result = null;

        try {
            HttpHeaders header = new HttpHeaders();

            header.add("Content-Type", Files.probeContentType(file.toPath()));
            result = new ResponseEntity<>(FileCopyUtils.copyToByteArray(file), header, HttpStatus.OK);
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        return result;
    }

    @PostMapping("/delete")
    @ResponseBody
    public ResponseEntity<String> deleteFile(String fileName, Long member_id) {

        File file;

        try {
            file = new File("c:\\resources\\" + URLDecoder.decode(fileName, "UTF-8"));

            file.delete();// 서버 측 썸네일 파일 삭제

            String largeFileName = file.getAbsolutePath().replace("s_", "");// s_를 제거하여 원본파일 가르키기

            file = new File(largeFileName);

            file.delete();// 원본 파일도 삭제

        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }

        attachDAO.delete(fileName.split("_")[1]);// delete(uuid) : _로 구분하여 2번째 값이 uuid이므로

        return new ResponseEntity<String>("deleted", HttpStatus.OK);

    }

    private String getFolder() {
        

        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

        Date date = new Date();// 지금 일시

        String str = sdf.format(date);// ex) 2020-01-01

        return str.replace("-", File.separator);// 2020\01\01
    }

    private boolean checkImageType(File file) {

        try {
            String contentType = Files.probeContentType(file.toPath());

            return contentType.startsWith("image");

        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

        return false;
    }
}
```

<br/>

> 결과 화면

1. 없는 상태

![](https://images.velog.io/images/sonchanwoo/post/0d2fb797-395c-4a64-a90a-09b5599620b4/image.png)

![](https://images.velog.io/images/sonchanwoo/post/7744ed64-7e49-4788-ae70-2df78c72bb63/image.png)

2. 업로드 후

![](https://images.velog.io/images/sonchanwoo/post/78ed8c8d-7614-42c3-9dd1-c48adc0862c6/image.png)

![](https://images.velog.io/images/sonchanwoo/post/2f23e790-0029-47b7-9d86-ab3999b530c4/image.png)

3. 삭제 후

![](https://images.velog.io/images/sonchanwoo/post/68f38570-b09a-4388-9401-ab8f6b69f2ba/image.png)