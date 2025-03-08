
- [[#heap dump 파일|heap dump 파일]]
	- [[#heap dump 파일#heap dump 파일은 무엇인가?|heap dump 파일은 무엇인가?]]
	- [[#heap dump 파일#heap dump 파일 생성 시기|heap dump 파일 생성 시기]]
- [[#OOM 발생 케이스|OOM 발생 케이스]]
	- [[#OOM 발생 케이스#OutOfMemoryError: Java Heap Space|OutOfMemoryError: Java Heap Space]]
	- [[#OOM 발생 케이스#OutOfMemoryError: GC Overhead limit exceeded|OutOfMemoryError: GC Overhead limit exceeded]]
	- [[#OOM 발생 케이스#OutOfMemoryError: Requested array size exceeds VM limit|OutOfMemoryError: Requested array size exceeds VM limit]]
	- [[#OOM 발생 케이스#OutOfMemoryError: Metaspace|OutOfMemoryError: Metaspace]]
	- [[#OOM 발생 케이스#OutOfMemoryError: unable to create native thread|OutOfMemoryError: unable to create native thread]]
- [[#heap dump 파일 생성 방법|heap dump 파일 생성 방법]]
	- [[#heap dump 파일 생성 방법#OOM이 발생하는 경우에 힙 덤프 파일 자동 생성|OOM이 발생하는 경우에 힙 덤프 파일 자동 생성]]
	- [[#heap dump 파일 생성 방법#실시간 스냅샷 생성|실시간 스냅샷 생성]]
- [[#OutOfMemory 발생하는 예제 구현|OutOfMemory 발생하는 예제 구현]]
- [[#힙덤프 파일 분석|힙덤프 파일 분석]]
	- [[#힙덤프 파일 분석#이클립스 MAT(Eclipse Memory Analyzer Tool)|이클립스 MAT(Eclipse Memory Analyzer Tool)]]
	- [[#힙덤프 파일 분석#VisualVM|VisualVM]]
	- [[#힙덤프 파일 분석#인텔리제이 사용|인텔리제이 사용]]
- [[#코드를 통한 OOM 예방|코드를 통한 OOM 예방]]
	- [[#코드를 통한 OOM 예방#불변 객체로 생성하라|불변 객체로 생성하라]]
	- [[#코드를 통한 OOM 예방#FileInputStream을 사용해라|FileInputStream을 사용해라]]
	- [[#코드를 통한 OOM 예방#Resource를 반납했는지 확인하세요|Resource를 반납했는지 확인하세요]]
	- [[#코드를 통한 OOM 예방#스트림을 사용해라|스트림을 사용해라]]
		- [[#스트림을 사용해라#충분한 메모리 확보|충분한 메모리 확보]]
	- [[#코드를 통한 OOM 예방#메모리 사이즈를 결정하자|메모리 사이즈를 결정하자]]
	- [[#코드를 통한 OOM 예방#young generation 영역을 좀더 확보하자|young generation 영역을 좀더 확보하자]]
	- [[#코드를 통한 OOM 예방#GC 로그를 남겨놓자|GC 로그를 남겨놓자]]
- [[#References|References]]


## heap dump 파일
### heap dump 파일은 무엇인가?
힙 메모리는 임의로 생성한 객체들이 동적으로 할당되는 공간을 의미하는데, **힙 덤프 파일은 운영중인 애플리케이션의 힙 메모리 영역을 스냅샷으로 기록한 내역을 저장한 파일을 의미**합니다.

### heap dump 파일 생성 시기
모든 경우에 대해서 스냅샷을 생성할 필요는 없고, 런타임 시점에 OOM(Out of Memory)이 발생하는 경우에 스냅샷을 생성하고 파일 내용을 분석하면 됩니다.

## OOM 발생 케이스
### OutOfMemoryError: Java Heap Space
- 힙 영역에 공간이 부족한 경우에 발생합니다. 가장 많이 확인되는 케이스일 것입니다.
- 힙 영역에 대한 공간 확보는 GC를 통해서 이루어지는데, GC를 수행하기 이전에 힙 메모리 영역이 부족할 경우에 발생합니다.

```java
public class JavaHeapSpace {
  public static void main(String[] args) throws Exception {
    String[] array = new String[100000 * 100000];
  }
}
```

### OutOfMemoryError: GC Overhead limit exceeded
- GC를 진행하는데 CPU 98% 이상 사용되고, GC 이후에 2% 미만으로 복구 되었을 경우입니다.
- 무분별하게 객체를 생성하는 경우에 발생합니다. 문제점을 해결하고, 경우에 따라서 메모리 사이즈를 늘리는 것을 추천합니다.
```java
public class GCOverhead {
  public static void main(String[] args) throws Exception {
    Map<Long, Long> map = new HashMap<>();
    for (long i = 0l; i < Long.MAX_VALUE; i++) {
      map.put(i, i);
    }
  }
}
```

### OutOfMemoryError: Requested array size exceeds VM limit
- 힙 영역보다 더 큰 영역의 배열을 할당할 경우 발생합니다.
- 배열 사이즈를 조정하거나 메모리 사이즈를 증가시켜서 해결할 수 있습니다.
```java
public class GCOverhead {
  public static void main(String[] args) throws Exception {
    for (int i = 0; i < 10; i++) {
      int[] arr = new int[Integer.MAX_VALUE - 1];
    }
  }
}
```

### OutOfMemoryError: Metaspace
- Metaspace 영역이 부족한 경우에 발생합니다.
- 클래스의 메타데이터(클래스 이름, 생성 정보, 필드 정보, 메서드 정보 등)가 저장되는 공간으로 JDK 7 이전에는 PermGen으로 정의되었습니다. java8 이후부터는 메타데이터는 메타스페이스에 저장됩니다. 메타스페이스는 Native Memory(운영체제에서 제공하는 메모리 공간, 힙 영역 외부에 존재함)에 저장됩니다.
- PermGen 영역은 힙 메모리에 포함되어 적은 범위로 설정되어 있기 때문에 GC가 빈번히 발생했으며, 이를 개선하고자 메타 스페이스 영역으로 변경되었습니다.
- **메타 스페이스 영역으로 개선되면서 힙 메모리 영역을 사용하지 않고 OS에서 제공하는 native 메모리 영역을 사용하므로 GC를 수행하지 않고도 자동으로 크기를 증가시켜서 공간을 확보할 수 있게 되었습니다.**
- 그래도 자동으로 크기는 증가하는 과정보다 더 많은 메타 데이터들이 저장되면, 에러가 발생할 수 있기 때문에 `-XX:MetaspaceSize`, `-XX:MaxMetaspaceSize` 설정을 추가하여 오류를 해결할 수 있습니다. (기본값은 20MB)
	- JVM 옵션의 `-XX:~`에서 XX는 extension option의 약자입니다.


### OutOfMemoryError: unable to create native thread
- 사용가능한 쓰레드가 존재하지 않는 경우에 발생합니다.
```java
public class ThreadsLimits {
  public static void main(String[] args) throws Exception {
    while (true) {
      new Thread(
          new Runnable() {
            @Override
            public void run() {
              try {
                Thread.sleep(1000 * 60 * 60 * 24);
              } catch (Exception ex) {}
            }
          }
      ).start();
    }
  }
}
```

## heap dump 파일 생성 방법
힙 덤프 파일을 생성하는 방법에는 2가지가 있습니다.
- 실시간으로 스냅샷 생성
- OOM(OutOfMemory)이 발생하는 시점에 자동 생성

### OOM이 발생하는 경우에 힙 덤프 파일 자동 생성
- 애플리케이션 내부적으로 OOM이 발생하는 경우에 힙 덤프 파일을 생성할 수 있도록 옵션을 추가할 수 있습니다.
```shell
java -jar -XX:+HeapDumpOnOutOfMemoryError \\
   -XX:HeapDumpPath=/home/centos/application/dumps/ \\
   -XX:OnOutOfMemoryError="kill -9 %p" \\
   application-0.0.1-SNAPSHOT.jar
```
- XX:+HeapDownOnOutOfMemoryError: OOM이 발생하는 경우에 힙덤프 파일을 생성합니다.
- XX:HeapDumpPath: 힙덤프가 생성되는 디렉토리 경로를 지정합니다.
- XX:OnOutOfMemoryError: OOM이 발생하는 경우 수행할 스크립트를 지정합니다. (보통 OOM이 발생하면 애플리케이션이 다운되기 때문에 재시작 스크립트를 다시 수행하기도 합니다.)

### 실시간 스냅샷 생성
- 사용자가 실시간으로 모니터링을 하다가 스냅샷을 생성할 수 있습니다.
- 스냅샷을 생성하기 위해서 실행중인 프로세스 아이디를 확인합니다.
```shell
ps -ef | grep java
``` 
![[Pasted image 20250307162053.png]]
위 실행 결과를 보면 PID가 1번인 것을 알수 있습니다. 현재 실행환경이 도커 컨테이너이기 때문에 PID가 1번으로 나옵니다.


- 호스트 운영체제에서 도커 컨테이너로 실행하는 경우에 다음과 같이 PID를 조회할 수 있습니다.
```shell
docker inspect --format '{{.State.Pid}}' fineAnts_app
```
![[Pasted image 20250307161252.png]]
  
**jmap 명령어로 힙덤프 파일 생성하기**
다음은 힙 덤프 파일을 생성하기 위한 명령어 형식입니다.
```shell
jmap -dump:format=b,file=testdump.hprof ${pid}
```

예를 들어 위 예시에서 나온 fineAnts_app spring 서버의 힙 덤프 파일을 생성해봅니다. testdump.hprof는 생성될 힙 덤프 파일 이름입니다.
```shell
jmap -dump:format=b,file=testdump.hprof 1
```
![[Pasted image 20250307162311.png]]
실행 결과를 보면 약 190MB 정도의 testdump.hprof 파일이 생성된 것을 볼수 있습니다.

## OutOfMemory 발생하는 예제 구현
다음과 같은 자바 소스코드를 구현하여 OOM이 발생할 수 있도록 구현해봅니다.

우선은 다음과 같이 예제 클래스를 생성합니다.
Example1.java
```java
import java.util.ArrayList;
public class Example1{
	public static void main(String[] args) throws InterruptedException{
		Example1 example = new Example1();
		example.test();
	}
	public void test() throws InterruptedException {
		ArrayList<int[]> list = new ArrayList();
		try {
			for(int i=0; i < 250000; i++) {
		      list.add(new int[10_000_000]); // 리스트에 배열을 추가한다
		      System.out.println(i);
		     Thread.sleep(1);
		    }
		  } catch (Exception e) {
		    e.printStackTrace();
		  }
	}
}
```

위 자바 소스 코드 파일을 컴파일합니다.
```shell
javac Example1.java
```

MANIFEST.MF 파일을 작성합니다.
MANIFEST.MF
```text
Manifest-Version: 1.0
Main-Class: Example1
```

jar 파일로 패키징합니다.
```
jar cvmf MANIFEST.MF Example1.jar Example1.class
```
![[Pasted image 20250307163756.png]]
위 실행 결과와 같이 Example1.jar 파일을 생성합니다.

jar 파일을 실행시켜서 오류를 발생시키고 힙 덤프 파일을 생성해봅니다.
```shell
java -jar -Xms128M -Xmx128M \
-XX:+HeapDumpOnOutOfMemoryError \
-XX:HeapDumpPath=./ \
-XX:OnOutOfMemoryError="kill -9 %p" \
Example1.jar
```
![[Pasted image 20250307164111.png]]

위 실행 결과를 보면 java_pid11486.hprof 힙 덤프 파일이 생성된 것을 볼수 있습니다.

## 힙덤프 파일 분석
### 이클립스 MAT(Eclipse Memory Analyzer Tool)
이클립스 MAT는 다음 링크를 통해서 다운로드받고 실행합니다.
- https://eclipse.dev/mat/download/

![[Pasted image 20250307164844.png]]


생성된 힙 덤프 파일을 이클립스 MAT을 이용해서 실행합니다.
![[Pasted image 20250307164919.png]]

다음 화면에서 Reports 메뉴에서 Leak Suspects 메뉴를 선택해봅니다.
![[Pasted image 20250307165027.png]]

보고서를 확인해보면 Example1 클래스의 test() 메서드에서 12번째 줄에서 에러가 발생하였고 Object[] 배열 타입의 인스턴스중에 하나가 메모리를 99.25% 점유하고 있는 것을 알려주고 있습니다.
![[Pasted image 20250307165122.png]]

위 화면에서 Details 버튼을 누르면 Dominator Tree를 확인할 수 있습니다.
![[Pasted image 20250307165601.png]]
실행 결과를 보면 int[] 타입의 배열 3개가 99.25%(120,000,048)을 메모리에서 차지하는 것을 볼수 있습니다.

다음 화면에서  "See stacktrace" 링크를 클릭하여 stacktrace를 통해서 에러가 발생하는 시작점을 확인할 수 있습니다.
![[Pasted image 20250307165819.png]]

실행 결과를 보면 Example1.java 파일에서 12번째 줄에서 에러가 발생한 것을 볼수 있습니다.
![[Pasted image 20250307165903.png]]

Example1.java 파일에서 12번째 줄의 코드는 다음과 같습니다. 실행 결과를 보면 리스트에 new int[10_000_000] 크기의 배열을 넣으려다가 OOM이 발생한 것을 볼수 있습니다.
![[Pasted image 20250307165943.png]]

### VisualVM
VisualVM에는 로컬에서 실행하고 있는 애플리케이션을 모니터링할 수 있고, 이미 생성된 힙 덤프 파일을 확인할 수 있습니다.
![[Pasted image 20250308113658.png]]

다음 화면을 보면 OutOfMemoryError가 어느 스레드에서 발생했는지 분석할 수 있습니다. 이 예제 같은 경우에는 main 스레드에서 발생한 것을 알수 있습니다.
![[Pasted image 20250308114138.png]]

스레드 탭을 보면 어느 코드 라인에서 어떤 객체에서 OOM이 발생했는지 확인할 수 있습니다. 이 예제 같은 경우는 Example1.java 파일에서 12번째 줄에서 OOM이 발생한 것을 확인할 수 있습니다.
![[Pasted image 20250308114422.png]]

### 인텔리제이 사용
인텔리제이 IDE를 이용하여 힙 덤프 파일을 열어보겠습니다.
![[Pasted image 20250308115042.png]]

용량을 제일 많이 차지하는 int[] 부분을 분석하면 Example1.java 파일에서 12번째 줄에서 OOM이 발생한 것을 볼수 있습니다.
![[Pasted image 20250308115104.png]]

## 코드를 통한 OOM 예방
- OOM은 다양한 케이스에서 발생할 수 있지만, 케이스를 확인해보면 무분별하게 객체를 생성하거나 rechable 상태를 유지할 경우에 발생하는게 대부분입니다.
- 그래서 코드렙레에서 주의를 기울여 작성하는게 OOM을 예방하기 위한 가장 최선의 방법입니다.

### 불변 객체로 생성하라
- 불변 객체는 객체의 내부 상태가 변경되지 않기 때문에 GC에서 reachable 상태인지 수시로 확인할 필요가 없으므로 GC의 부담을 줄일 수 있습니다.
- 불변 객체는 쓰레드 세이프하기 때문에 동시성 문제도 피할 수 있습니다.

### FileInputStream을 사용해라
- `FileInputStream`을 이용하여 파일 처리시 메모리 사용량을 효율적으로 관리하고 OOM 오류를 방지할 수 있습니다.
```java
try (FileInputStream fis = new FileInputStream("your_file_path")) {
    byte[] buffer = new byte[4096]; // 또는 적절한 크기로 조정
    int bytesRead;
    while ((bytesRead = fis.read(buffer)) != -1) {
        // buffer를 이용한 작업 수행
    }
} catch (IOException e) {
    // 예외 처리
}
```

### Resource를 반납했는지 확인하세요
- Java에서 `finally` 블록을 사용해서 스트림을 명시적으로 닫는 것은 메모리 관리와 리소스 관리에 도움이 됩니다.
- JDK8 이후부터는 `try-with-resources 구문으로 대체할 수 있습니다. 이 구문을 사용하여 리소스 누수를 방지할 수 있습니다.
```java
public class StreamExample {
    public static void main(String[] args) {
        try (FileInputStream fis = new FileInputStream("file.txt")) {
            // 스트림을 이용하여 파일 읽기 작업 수행
        } catch (IOException e) {
            // 예외 처리
        }
    }
}
```

### 스트림을 사용해라
- Stream API는 함수형 프로그래밍 스타일을 지원하며, 중간 연산과 최종 연산을 사용하여 데이터를 스트림으로 처리하여 대용량 데이터를 효율적으로 처리할 수 있습니다.
- Stream API가 OOM을 해결하는데 도움이 되는 특징
	- Lazy Evaluation : 중간 연산들을 즉시 처리하지 않고, 최종 연산이 호출할 때 데이터를 처리하기 때문에 대용량 데이터를 한번에 모두 메모리에 적재하지 않고, 처리할 수 있습니다.
	- Pipelining : Stream API는 연속적인 연산들을 파이프라이닝하여 데이터를 순차적으로 처리하기 때문에 중간 연산들에 대한 결과를 임시적으로 저장하지 않고도 메모리 사용을 최적화할 수 있습니다.
	- Parallel Processing : 적절한 상황에서 데이터를 여러개의 스레드로 나누어 병렬 처리가 가능합니다.

#### 충분한 메모리 확보
- 코드를 OOM이 발생하지 않도록 작성하는 것도 중요하지만, 애플리케이션 가용 범위 내에서 인프라 리소스를 적절하게 사용하고 있는지 확인해봐야 합니다.

### 메모리 사이즈를 결정하자
- 애플리케이션 실행 시 최소/최대 메모리 사이즈를 설정하면 좋습니다.
	- Xmx: 최대 메모리 사이즈
	- Xms: 초기 메모리 사이즈
- 만약 별도 설정하지 않으면, 최대 메모리 사이즈는 서버의 가용 메모리 1/4이고, 최소 메모리 사이즈는 서버의 가용 메모리 1/64 정도로 설정됩니다.


> [!NOTE] 만약 메모리가 8GB의 인스턴스에서 아무런 설정 없이 애플리케이션을 운영하면?
> 최대 메모리 사이즈는 8GB / 4 = 2GB이고, 초기 메모리 사이즈는 8GB / 64 = 127MB


- 실제로 운영되는 애플리케이션의 메모리 사이즈를 확인하기 위해서는 `java -XX:+PrintFlagsFinal -version 2>&1 | grep -i -E 'heapsize|metaspacesize|version` 명령어를 실행하면 현재 설정된 메모리 사이즈를 확인할 수 있습니다.
![[Pasted image 20250308124018.png]]
위 실행 결과를 보면 MaxHeapSize는 249,561,088byte로써 238MB 크기입니다. 그리고 초기 힙 사이즈는 16,777,216byte로써 16MB입니다.

- 최대/최소 메모리 사이즈를 결정하는 기준은 딱히 없습니다. 일반적으로 인스턴스에 애플리케이션 하나만 운영된다면, 전체 리소스의 1/2정도로 측정하고, 최대/최소 메모리는 같게 설정합니다.


> [!NOTE] 최대/최소 메모리 사이즈는 같아야 할까?
> 최대/최소 메모리 사이즈는 같아야 합니다. 최소 메모리 사이즈는 초기 메모리 사이즈를 의미합니다. 초기 메모리 사이즈에서 어느 정도 메모리가 가득차면 GC가 발생하고, 메모리 사이즈를 조금씩 올리는 형태(GC 수행후에도 메모리 사이즈가 부족하면 증가하게됨)로 최대 메모리 사이즈까지 증가하게 됩니다. 그래서 초기에 최대 메모리 사이즈까지 설정하면 그만큼 GC가 덜 발생하게 됩니다. 그렇다고 하더라도 너무 크게 사이즈를 설정하면 한번 GC가 발생할 때 부하가 크게 발생할 수 있으니 적절한 사이즈를 알아보는것이 좋습니다.

### young generation 영역을 좀더 확보하자
- STW(Stop-The-World, GC 실행시 애플리케이션의 모든 스레드가 멈추는 현상)의 발생은 Young Gen -> Old Gen으로 옮기는 과정인 Major GC에서 발생합니다.
- Yong Gen 영역에서 발생하는 Minor GC를 적극적으로 활용하면 STW를 줄일 수 있습니다.
- Yong Gen은 Old Gen의 2배로 설정하는 것이 효율적입니다. (-XX:NewRadio=2)
- Eden과 Survivor의 비율을 8:1로 설정한다. (-XX:SurvivorRatio=8)

### GC 로그를 남겨놓자
- GC 이력을 로그 파일로 남겨 놓을 수 있습니다.
- `Xloggc` 옵션으로 필요한 정보를 남길 수 있도록 하자.
- `-Xloggc:gc-%t.log`: 로깅할 파일을 지정합니다.
- `-XX:+PrintGCDetails`: GC 상세 내역을 기록합니다.(JDK 11 이후에는 -Xlog:gc=info 파라미터만 추가)
- `-XX:+PrintGCDateStamps`: GC 발생 시간을 기록합니다.(JDK 11 이후에는 decorator 옵션으로 변경)
```shell
java -jar -Xlog:gc=info:gc-%t.log:time
   -XX:HeapDumpPath=./  
   -XX:OnOutOfMemoryError="kill -9 %p" 
   -XX:+HeapDumpOnOutOfMemoryError 
   ./application-SNAPSHOT.jar
```

Example1.jar 파일을 기준으로 실행하여 GC 로그를 생성해보겠습니다.
```shell
java -jar -Xlog:gc=info:gc-%t.log:time \
   -XX:HeapDumpPath=./ \
   -XX:OnOutOfMemoryError="kill -9 %p" \
   -XX:+HeapDumpOnOutOfMemoryError \
   ./Example1.jar
```
![[Pasted image 20250308133846.png]]

## References
- https://incheol-jung.gitbook.io/docs/q-and-a/java/heap-dump-feat.-oom