
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




## 원인
### Problem Suspect 1
![](refImg/Pasted%20image%2020251210144630.png)



## 해결 방법



