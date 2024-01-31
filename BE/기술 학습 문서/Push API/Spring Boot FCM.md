
## 1. Admin SDK
스프링 서버에서 Firebase와 상호작용하기 위해서는 먼저 Admin SDK를 추가해야 합니다.

Firebase 프로젝트의 웹 앱 설정에서 서비스 계정 메뉴로 들어가서 다음과 괕이 새 비공개 키 생성 버튼을 클릭합니다.
![[Pasted image 20240131142556.png]]
![[Pasted image 20240131142601.png]]

새 비공개 키 생성 버튼을 누르면 설정 정보가 담긴 json 파일이 다운로드 받을 것입니다. 해당 파일은 추후 resources 디렉토리에 저장되어 사용됩니다.

그 다음에 Spring Boot 프로젝트에 의존성을 추가합니다.
```
// firebase  
implementation 'com.google.firebase:firebase-admin:9.1.1'
```

새 비공개 키로 생성한 json 파일을 resources 디렉토리 하위에 저장합니다.
![[Pasted image 20240131145601.png]]

스프링 빈 설정 클래스를 정의합니다.
```java
@SpringBootApplication  
public class FirebaseNotificationApplication {  
  
    @Bean  
    public FirebaseMessaging firebaseMessaging() throws IOException {  
       GoogleCredentials googleCredentials = GoogleCredentials  
          .fromStream(new ClassPathResource("secret/firebase/fineants-adminsdk.json").getInputStream());  
       FirebaseOptions firebaseOptions = FirebaseOptions.builder()  
          .setCredentials(googleCredentials)  
          .build();  
       FirebaseApp app = FirebaseApp.initializeApp(firebaseOptions, "my-app");  
       return FirebaseMessaging.getInstance(app);  
  
    }  
  
    public static void main(String[] args) {  
       SpringApplication.run(FirebaseNotificationApplication.class, args);  
    }  
}
```

### Push Notification 서비스 코드 구현

NotificationMessage.java
```java
@Getter  
@RequiredArgsConstructor  
@ToString  
public class NotificationMessage {  
    private String recipientToken;  
    private String title;  
    private String body;  
    private String image;  
    private Map<String, String> data;  
}
```

FirebaseMessagingService.java
```java
@Service  
@RequiredArgsConstructor  
public class FirebaseMessagingService {  
    private final FirebaseMessaging firebaseMessaging;  
  
    public String sendNotificationByToken(NotificationMessage notificationMessage){  
       Notification notification = Notification  
          .builder()  
          .setTitle(notificationMessage.getTitle())  
          .setBody(notificationMessage.getBody())  
          .setImage(notificationMessage.getImage())  
          .build();  
  
       Message message = Message  
          .builder()  
          .setToken(notificationMessage.getRecipientToken())  
          .setNotification(notification)  
          .putAllData(notificationMessage.getData())  
          .build();  
  
       try{  
          firebaseMessaging.send(message);  
          return "Success Sending Notification";  
       }catch (FirebaseMessagingException e){  
          e.printStackTrace();  
          return "Error Sending Notification";  
       }  
    }  
}
```

NotificationRestController.java
```java
@RestController  
@RequestMapping("/notification")  
@RequiredArgsConstructor  
public class NotificationRestController {  
    private final FirebaseMessagingService firebaseMessagingService;  
  
    @PostMapping  
    public String sendNotificationByToken(@RequestBody NotificationMessage notificationMessage){  
       return firebaseMessagingService.sendNotificationByToken(notificationMessage);  
    }  
}
```


### Push Notification 테스팅
스프링 부트 서버 및 리액트 애플리케이션을 실행시키고 Postman에 다음과 같이 실행하여 스프링 부트 서버로 요청을 전송하여 리액트 클라이언트 앱이 알림 메시지를 받을 수 있도록 테스팅해봅니다.
![[Pasted image 20240131150102.png]]

위 화면에서 receipentToken은 리액트 클라이언트 앱이 받은 token의 값을 복사하여 넣습니다.
![[Pasted image 20240131150146.png]]

![[Pasted image 20240131150335.png]]
위 화면은 포스트맨의 실행 결과입니다.

만약 리액트 클라이언트 앱이 백그라운드 상태라면 다음과 같이 알림이 오는 것을 확인할 수 잇습니다. (OS에서 별도의 크롬 알림 허용 설정이 되어 있어야 합니다.)
![[Pasted image 20240131150349.png]]

