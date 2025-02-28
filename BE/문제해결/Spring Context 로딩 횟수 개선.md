
## 배경
- 통합 테스트 및 컨트롤러 테스트를 수행할 때 Spring Context 로딩 횟수를 체크하였습니다.
- 통합 테스트의 경우 Spring Context를 20회 로딩하는 것을 확인하였습니다.
- 컨트롤러 테스트의 경우에는 13회 로딩합니다.

## 테스트 수행 중 Spring Context를 재로딩하는 이유는 무엇인가?
- 테스트 실행시 `@MockBean`, `@SpyBean` 등의 목 객체 주입으로 인해서 스프링 빈을 다시 구성해야 하는 경우 Spring Context를 다시 로딩해야 합니다. 즉, 테스트 코드에 목빈 관련 애노테이션 사용시 Spring Context가 재로딩됩니다.

## 개선방향
 `@TestConfiguration`을 선언한 클래스를 생성한 다음에 해당 클래스에 스프링 빈 목 객체를 선언합니다.
```java
@TestConfiguration  
public class TestConfig {  
    @MockBean  
    private AmazonS3Service mockedAmazonS3Service;  
  
    @MockBean  
    private VerifyCodeManagementService verifyCodeManagementService;  
  
    @MockBean  
    private VerifyCodeGenerator verifyCodeGenerator;
    // ...
}
```

TestConfig 클래스를 참조하기 위해서 `@ContextConfiguration` 애노테이션을 사용하여 가져옵니다.
```java
@ActiveProfiles("test")  
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)  
@ContextConfiguration(classes = {AmazonS3TestConfig.class, TestConfig.class})    
// ...
public abstract class AbstractContainerBaseTest {
  // ...
}
```

목 객체를 사용하고자 하는 테스트 클래스에서는 `@Autowired` 애노테이션을 사용하여 필드로 주입받은 다음에 모킹 처리를 하면 됩니다.
```java
class MemberServiceTest extends AbstractContainerBaseTest {

	@Autowired  
	private AmazonS3Service mockAmazonS3Service;

	// ...
}
```

## 개선 결과
- 기존 전체 테스트 수행 시간이 5분에서 4분 23초로 단축되었습니다. 약 12.3% 성능 향상되었습니다.
