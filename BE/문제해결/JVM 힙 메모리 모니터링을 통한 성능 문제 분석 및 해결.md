
## 배경
VisualVM의 Monitor 기능을 통해서 Spring 서버의 힙 메모리 사용량이 계속 증가하다가 떨어지는 것을 모니터링하였습니다. 다음 결과는 로컬 서버에서 종목의 현재가 스케줄러를 활성화한 상태에서 모니터링한 결과입니다.

**Monitor 실행 결과**
![](refImg/Pasted%20image%2020251216141808.png)

힙 덤프 파일을 생성한 다음에 Eclipse Memory Analyzer 도구를 이용하여 히스토그램(Histogram)을 분석합니다. 히스토그램을 분석함으로써 전체 힙 메모리 중에서 가장 많은 공간을 차지하는 클래스가 무엇인지 파악합니다.
실행 결과를 보면 `byte[]` 배열이 49% 정도(Shallow Heap)를 차지하고 있습니다.
![](refImg/Pasted%20image%2020251216155333.png)

`byte[]` 배열을 참조하고 있는 상위 객체를 탐색해봅니다.
![](refImg/Pasted%20image%2020251216155456.png)

다음 실행 결과를 보면 8개의 reactor-http-nio 스레드가 대략 62% 정도(Ref. Shallow Heap)의 메모리 공간을 차지하고 있는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251216155541.png)

그러면 reactor-http-nio 스레드가 발생하게된 메서드를 추적해봅니다. 그러기 위해서는 VisualVM 툴을 이용하여 프로파일링할때 Profile classes에 `reactor.netty.**` 정규식을 추가합니다.
![](refImg/Pasted%20image%2020251216160314.png)

CPU 프로파일링을 해본 결과 우선 첫번째로 scheduling-1이 기록되었고 해당 스케줄링 작업은 KisProductionScheduler 클래스에 의해서 refreshCurrentPrice 메서드가 실행된 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251216160839.png)

refreshCurrentPrice 메서드 상세 내용을 추적하면 다음과 같습니다. 분석 결과 내부적으로 다시 KisService 객체를 이용하여 refreshAllStockCurrentPrice 메서드를 수행합니다.
![](refImg/Pasted%20image%2020251216161231.png)

그리고 reactor-http-nio 스레드 부분을 보면 내부적으로 KisClient 객체를 이용하여 fetchCurrentPrice 메서드 호출하는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251216161418.png)

위와 같은 메모리 및 CPU 프로파일링 분석을 통해서 힙 메모리 사용량이 증가된 원인은 다음과 같습니다.
- KisProductionScheduler 클래스의 refreshCurrentPrice 메서드의 빈번한 실행
	- 현재 장시간동안 5초에 한번씩 실행중
- WebClient 사용(비동기 I/O)
	- 현재가 갱신 과정에서 WebClient를 사용하여 외부 API와 지속적으로 통신함
	- WebClient는 Reactor Netty를 기반으로 하며, 이 과정에서 `byte[]` 형태의 I/O 버퍼와 `reactor-http-nio` 스레드를 사용하게 됨

---


확인된 메모리 누수 및 비효율 증거
- I/O 버퍼 누수
	- `byte[]` 배열이 Shallow Heap의 49% 점유
	- WebClient를 통해 받은 응답 스트림의 버퍼가 `reactor-http-nio` 스레드에 의해서 명시적으로 해제되지 않고 남아있음
- 스레드 및 컨텍스트 누수
	- `reactor-http-nio` 스레드 8개가 `Ref. Shallow Heap`의 62% 점유
	- I/O 작업을 담당하는 Reactor Netty의 이벤트 루프 스레드가 활성 상태(GC Root)를 유지하며, 누수된 I/O 버퍼를 붙잡고 있음
- 로깅/AOP 관련 누수
	- `ConcurrentHashMap$Node`와 ShadowMatchImpl 객체의 간접적인 참조 누수
	- `lettuce-eventExecutorLoop` 스레드와 **Logback `LoggerContext`**의 참조 누수 경로가 확인되었으며, 이는 **클래스 로더 누수**와 연관될 가능성이 높음

---

## 로깅 AOP 관련 누수
### 배경
Eclipse Memory Analyzer 도구의 메모리 누수 의심 보고서 결과는 다음과 같습니다.
![](refImg/Pasted%20image%2020251218113347.png)

첫번째 메모리 누수 의심은 `byte[]` 배열의 데이터가 30.97%를 점유하고 있고 `Object[]` 배열 하나에서 대부분을 참조하고 있습니다. 그리고 이러한 `Object[]` 배열을 Monitor Ctrl-Break 스레드가 참조하고 있습니다.
![](refImg/Pasted%20image%2020251218113356.png)

세번째 메모리 누수 의심은 AspectJExpressionPointcut 인스턴스가10.35% 메모리 점유하고 있습니다. 해당 인스턴스들은 대부분 `ConcurrentHashMap$Node[]` 배열에서 참조하고 있습니다. 그리고 이러한 배열 데이터는 DefaultListableBeanFactory에 의해서 참조되고 있습니다.
![](refImg/Pasted%20image%2020251218113559.png)

힌트를 보면 메모리 누수 1, 3은 서로 연관되어 있을 수 있습니다. 왜냐하면 참조 체인이 공통적인 시작을 가지고 있기 때문이라고 합니다.
![](refImg/Pasted%20image%2020251218113723.png)

## 원인
### AOP 포인트컷의 과도한 캐싱 (Suspect 3)
- 현상 : 8개의 `AspectJExpressionPointcut`이 약 11MB를 차지하고 있습니다.
- 이유 : Spring AOP는 포인트컷(로그를 찍을 대상)을 정의하면, 해당 식에 맞는 메서드들을 찾아내기 위해 애플리케이션의 모든 클래스를 분석합니다.
- 결과 : 특히 5초 간격으로 돌아가는 스케줄러와 관련된 메서드들이 이 포인트컷에 포함되어 Spring은 해당 메서드를 호출할때마다 매칭 여부를 확인하고, 결과를 `ConcurrentHashMap`에 캐싱합니다. 이 과정에서 생성된 포인트컷 메타데이터가 `DefaultListableBeanFactory`에 쌓이게 됩니다.

### IntelliJ 콘솔 버퍼의 누적 (Suspect 1)
- 현상 : `byte[]`가 31%를 차지하며 IntellJ 스레드(Monitor Ctrl-Break)가 이를 붙잡고 있습니다.
- 이유 : 로그 AOP가 실행될때마다 표준 출력(Console)으로 로그를 내보냅니다.
- 결과 : 5초마다 수많은 종목의 현재가를 갱신하면서 발생하는 **방대한 양의 로그 메시지**가 IntelliJ의 콘솔 버퍼(`byte[]`)에 차곡차곡 쌓인것입니다. 힌트에서 말한 "공통의 시작점"은 바로 이 **로그를 생성하는 주체(AOP)와 로그를 수집하는 주체(IntelliJ 모니터링 스레드)** 가 하나의 실행 프로세스 안에 묶여 있음을 뜻합니다.

### Metaspace 고갈 및 톱니형 패턴
- Metaspace : AOP는 기존 클래스를 감싸는 프록시(Proxy) 클래스를 런타임에 동적으로 생성합니다. 스케줄러가 빈번하게 작동하며 AOP가 개입할수록 더 많은 프록시 관련 메타데이터가 Metaspace에 기록되어 100%에 도달한 것입니다.
- 톱니형 패턴 : 스케줄러가 돌아갈 때마다 AOP 로직이 실행되며, 엄청난 양의 임시 객체(로그 문자열, 포인트 컷 분석 객체)를 생성합니다. 이것이 힙을 채우다 GC에 의해 지워지는 과정이 반복되며 톱니 모양을 그리게 됩니다.

## 해결방법
### 포인트컷(Pointcut) 범위 좁히기
현재 설정된 로그 기록 포인트컷은 다음과 같습니다.
- service 패키지 밑에 있는 모든 메서드를 대상으로 메서드의 실행 전후에 로그를 작성하고 있습니다.
- 해당 포인트컷 설정은 로그의 유용성이 없다고 판단하여 해당 AOP는 제거할 예정
```java
@Component  
@Aspect  
@Slf4j  
@Profile("!test")  
public class ServiceLogAspect {  
    private long startTime;  
  
    // service의 모든 메서드에 대해 적용  
    @Pointcut("execution(* co.fineants..service.*.*(..))")  
    public void pointCut() {  
  
    }  
  
    // 메서드 호출 전 로그 남기기  
    @Before("pointCut()")  
    public void logBefore(JoinPoint joinPoint) {  
       startTime = System.currentTimeMillis();  
       String methodName = ((MethodSignature)joinPoint.getSignature()).getMethod().getName();  
       String args = Arrays.toString(joinPoint.getArgs());  
       log.info("Entering Service: Method={} with Args={}", methodName, args);  
    }  
  
    // 메서드 호출 후 정상적으로 반환된 경우 로그 남기기  
    @AfterReturning(pointcut = "pointCut()", returning = "result")  
    public void logAfterReturning(JoinPoint joinPoint, Object result) {  
       String methodName = ((MethodSignature)joinPoint.getSignature()).getMethod().getName();  
       log.info("Exiting Service: Method={}, with Return={}", methodName, result);  
    }  
  
    // 완전히 종료된후 메서드 실행시간 측정하기  
    @After("pointCut()")  
    public void logAfter(JoinPoint joinPoint) {  
       long executionTime = System.currentTimeMillis() - startTime;  
       String methodName = ((MethodSignature)joinPoint.getSignature()).getMethod().getName();  
       log.info("Method={}, ExecutionTime={}ms", methodName, executionTime);  
    }  
}
```

### 로그 출력 방식 변경
- 표준 출력(System.out)은 IntelliJ와 같은 IDE 환경에서 메모리를 점유합니다.
- 로그 레벨을 DEBUG에서 INFO로 높여서 불필요한 출력을 줄이거나 로컬 실행시에는 파일 로그 출력을 끕니다.
- 현재 이 환경은 로컬 개발 환경에서 표준 출력하는 것이기 때문에 배포 환경에서는 파일 로그만 출력됩니다.
- 로컬 개발 환경에서는 콘솔 로그 출력만 하도록 유지하고, 배포 환경에서는 동일하게 유지합니다.
![](refImg/Pasted%20image%2020251218124102.png)

## 실행 결과 확인
다음 결과는 포인트컷 클래스를 비활성화하고 로그 출력 방식을 변경한 다음에 메모리 누수 의심 기능을 실행한 결과입니다.
실행 결과를 보면 Suspect 3에 해당하는 **AspectJExpressionPointcut** 인스턴스의 메모리 점유가 사라진 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251218130016.png)


```
PROJECT_VERSION="0.0.32"
IMAGE_TAG="fineants/fineants-app:local-0.0.32"

docker build --platform linux/amd64 --build-arg PROJECT_VERSION="$PROJECT_VERSION" -t $IMAGE_TAG .
```