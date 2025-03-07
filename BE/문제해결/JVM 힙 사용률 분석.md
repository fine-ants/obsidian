
## 힙 사용율 모니터링
프로메테우스를 통하여 JVM의 힙 사용율을 모니터링 하였습니다. 그라파나 대시보드를 통하여 모니터링 한 결과, 힙 사용율은 85.09%를 사용하고 있습니다.
![[Pasted image 20250306123233.png]]

JVM의 힙을 분석해서 어느 지점에서 85%를 사용하고 있는지 알아보려고 합니다.

curl 명령어를 이용해서 서버의 actuator/heapdump 경로에 접근하여 파일을 다운로드 받습니다.
```
curl -u username:password -o heapdump.hprof "https://services.fineants.co/actuator/heapdump"
```

다운로드 받은 힙 덤프 파일을 확인해봅니다.
```shell
ls -lh heapdump.hprof
```
![[Pasted image 20250307143436.png]]

힙 덤프 파일을 열기 위해서는 다양한 방법이 있습니다. 여기서는 인텔리제이를 통해서 열어보겠습니다.
![[Pasted image 20250307143525.png]]
실행 결과를 보면 힙 덤프 파일의 내용을 모니터링 할 수 있습니다.

