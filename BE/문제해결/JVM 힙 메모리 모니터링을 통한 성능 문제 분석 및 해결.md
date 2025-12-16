
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
	

## 원인
### Problem Suspect 1
![](refImg/Pasted%20image%2020251210144630.png)



## 해결 방법



