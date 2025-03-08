
- [[#힙 사용율 모니터링|힙 사용율 모니터링]]
- [[#힙 덤프 파일 다운로드 및 열기|힙 덤프 파일 다운로드 및 열기]]
- [[#힙 덤프 파일을 이용한 Leak Suspects|힙 덤프 파일을 이용한 Leak Suspects]]
	- [[#힙 덤프 파일을 이용한 Leak Suspects#Problem Suspect 1|Problem Suspect 1]]
		- [[#Problem Suspect 1#1. AspectJExpressionPointcut 객체들의 메모리 사용|1. AspectJExpressionPointcut 객체들의 메모리 사용]]
		- [[#Problem Suspect 1#2. ConcurrentHashMap 사용|2. ConcurrentHashMap 사용]]
		- [[#Problem Suspect 1#3. RequestMappingHandlerAdapter의 메모리를 사용|3. RequestMappingHandlerAdapter의 메모리를 사용]]
		- [[#Problem Suspect 1#4. Thread 상태 분석|4. Thread 상태 분석]]
		- [[#Problem Suspect 1#5. 문제의 핵심|5. 문제의 핵심]]
		- [[#Problem Suspect 1#해결 방안|해결 방안]]
		- [[#Problem Suspect 1#결론|결론]]


## 힙 사용율 모니터링
프로메테우스를 통하여 JVM의 힙 사용율을 모니터링 하였습니다. 그라파나 대시보드를 통하여 모니터링 한 결과, 힙 사용율은 85.09%를 사용하고 있습니다.
![[Pasted image 20250306123233.png]]

JVM의 힙을 분석해서 어느 지점에서 85%를 사용하고 있는지 알아보려고 합니다.

## 힙 덤프 파일 다운로드 및 열기
curl 명령어를 이용해서 서버의 actuator/heapdump 경로에 접근하여 파일을 다운로드 받습니다.
```
curl -u username:password -o heapdump.hprof "https://services.fineants.co/actuator/heapdump"
```

다운로드 받은 힙 덤프 파일을 확인해봅니다.
```shell
ls -lh heapdump.hprof
```
![[Pasted image 20250307143436.png]]

힙 덤프 파일을 열기 위해서는 다양한 방법이 있습니다. 여기서는 이클립스 MAT를 통해서 엽니다.
![[Pasted image 20250308150900.png]]


## 힙 덤프 파일을 이용한 Leak Suspects
이클립스 MAT를 이용해서 힙 덤프 파일을 열게 되면 보고서 종류중에 Leak Suspects 보고서를 선택하여 메모리 누수가 의심되는 객체들을 자동으로 탐지하여 분석해봅니다.

다음 실행 결과를 보면 총 77.7MB 중에서 35.2 MB(약 45%)가 메모리 누수가 의심되고 있습니다.
![[Pasted image 20250308151530.png]]

### Problem Suspect 1
Problem Suspect 1의 내용은 다음과 같습니다.
```
8 instances of **org.springframework.aop.aspectj.AspectJExpressionPointcut**, loaded by **org.springframework.boot.loader.LaunchedURLClassLoader @ 0xf63969c0** occupy **10,627,176 (13.04%)** bytes. 

Biggest instances:

- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6dffe60 - 1,310,984 (1.61%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0dad0 - 1,280,864 (1.57%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0dbf8 - 1,283,808 (1.57%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0dd08 - 1,285,624 (1.58%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0de18 - 1,310,496 (1.61%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0df10 - 1,310,984 (1.61%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0e008 - 1,310,984 (1.61%) bytes. 
- org.springframework.aop.aspectj.AspectJExpressionPointcut @ 0xf6e0e118 - 1,533,432 (1.88%) bytes. 

Most of these instances are referenced from one instance of **java.util.concurrent.ConcurrentHashMap$Node[]**, loaded by **<system class loader>**, which occupies **455,856 (0.56%)** bytes. The instance is referenced by **org.springframework.beans.factory.support.DefaultListableBeanFactory @ 0xf66aa910**, loaded by **org.springframework.boot.loader.LaunchedURLClassLoader @ 0xf63969c0**. 

Thread **org.apache.tomcat.util.threads.TaskThread @ 0xfa672a50 https-jsse-nio-443-exec-10** has a local variable or reference to **org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter @ 0xf9d7e1d8** which is on the shortest path to **java.util.concurrent.ConcurrentHashMap$Node[1024] @ 0xf97299c0**. The thread **org.apache.tomcat.util.threads.TaskThread @ 0xfa672a50 https-jsse-nio-443-exec-10** keeps local variables with total size **9,920 (0.01%)** bytes.

Significant stack frames and local variables

- org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(Ljakarta/servlet/http/HttpServletRequest;Ljakarta/servlet/http/HttpServletResponse;Lorg/springframework/web/method/HandlerMethod;)Lorg/springframework/web/servlet/ModelAndView; (RequestMappingHandlerAdapter.java:892)
    - org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter @ 0xf9d7e1d8 retains 4,656 (0.01%) bytes
- org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(Ljakarta/servlet/http/HttpServletRequest;Ljakarta/servlet/http/HttpServletResponse;Lorg/springframework/web/method/HandlerMethod;)Lorg/springframework/web/servlet/ModelAndView; (RequestMappingHandlerAdapter.java:798)
    - org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter @ 0xf9d7e1d8 retains 4,656 (0.01%) bytes

The stacktrace of this Thread is available. [See stacktrace](pages/31.html). [See stacktrace with involved local variables](pages/32.html).

**Keywords**

- org.springframework.aop.aspectj.AspectJExpressionPointcut
- org.springframework.boot.loader.LaunchedURLClassLoader
- java.util.concurrent.ConcurrentHashMap$Node[]
- org.springframework.beans.factory.support.DefaultListableBeanFactory
- org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(Ljakarta/servlet/http/HttpServletRequest;Ljakarta/servlet/http/HttpServletResponse;Lorg/springframework/web/method/HandlerMethod;)Lorg/springframework/web/servlet/ModelAndView;
- RequestMappingHandlerAdapter.java:892
- org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(Ljakarta/servlet/http/HttpServletRequest;Ljakarta/servlet/http/HttpServletResponse;Lorg/springframework/web/method/HandlerMethod;)Lorg/springframework/web/servlet/ModelAndView;
- RequestMappingHandlerAdapter.java:798
```
이 보고서는 Java 힙 덤프 또는 메모리 분석을 기반으로 spring 애플리케이션에서 메모리 사용 현황을 제공합니다. 특히 Spring AOP와 관련된 AspectJExpressionPointcut 객체들이 메모리를 많이 차지하고 있는 상황에 대한 분석입니다. 

#### 1. AspectJExpressionPointcut 객체들의 메모리 사용
- `org.springframework.aop.aspectj.AspectJExpressionPointcut` 는 Spring AOP의 AspectJ 표현식을 처리하는 클래스입니다. 이 클래스는 AOP 적용 시점을 정의하기 위해 사용됩니다.
- 8개의 인스턴스가 총 10,627,176 byte(10.1 MB, 13.04%)의 메모리를 차지하고 있다고 나와 있습니다. 그중 가장 큰 인스턴스들은 각각 약 1,280,000(1.2MB)~1,533,000(1.4MB) 바이트의 메모리를 사용하고 있습니다.
	- AspectJExpressionPointcut 객체는 AOP의 Advice(전후 처리 로직)와 JointPoint(메서드 호출 시점)을 결합하는데 사용됩니다.
	- 많은 메모리 사용은 여러 AspectJ 표현식이 메모리에 로드되었기 때문일 가능성이 큽니다.

#### 2. ConcurrentHashMap 사용
- **java.util.concurrent.ConcurrentHashMap$Node[]** 는 `ConcurrentHashMap`는 내부에서 사용되는 배열로, AOP 관련 AspectJExpressionPointcut 객체들이 `ConcurrentHashMap`에 저장되어 있는 구조로 보입니다.
- 이 `ConcurrentHashMap`은 Spring의 빈(Bean) 팩토리인 `DefaultListableBeanFactory`에 의해서 참조되고 있으며, 이로 인해 메모리에서 상단한 크기를 차지하고 있습니다.
	- **`ConcurrentHashMap$Node[]`** 은 455,856 바이트(0.56%)의 메모리를 사용하고 있습니다. 이는 `AspectJExpressionPointcut` 객체들을 캐시하거나 관리하는 데 사용되는 데이터 구조일 수 있습니다.

#### 3. RequestMappingHandlerAdapter의 메모리를 사용
- **RequestMappingHandlerAdapter**는 Spring MVC에서 HTTP 요청을 핸들링하는 클래스입니다. 이 클래스는 HTTP 요청을 적절한 핸들러 메서드와 매핑하여 실행하는 역할을 수행합니다.
- 이 객체는 HTTP 요청 처리 중, `HandlerMethod` 와 함께 메서드 호출 및 결과 모델을 관리합니다. 보고서에서 이 클래스는 4,656 바이트(0.01%) 메모리를 유지한다고 언급하고 있습니다.

#### 4. Thread 상태 분석
- 보고서에 나오는 **TaskThread** 는 Tomcat에서 실행 중인 HTTP 요청 처리 스레드입니다. 이 스레드는 `RequestMappingHandlerAdapter`를 사용하여 요청을 처리하고 있으며, TaskThread는 `ConcurrentHashMap$Node[]`와 관련 있는 `RequestMappingHandlerAdapter` 객체를 참조하고 있습니다.
- 이 TaskThread는 9,920 바이트의 메모리를 유지하며, 이는 스레드 로컬 변수나 참조가 차지하는 메모리입니다.

#### 5. 문제의 핵심
- 메모리를 많이 차지하는 `AspectJExpressionPointcut` 인스턴스들이 AOP와 관련된 클래스들로, 이러한 객체들이 메모리 사용을 증가시키고 있습니다. 특히 AOP에서 사용하는 AspectJ 표현식을 처리하는 데 많은 메모리가 소모되고 있습니다.
- `ConcurrentHashMap`은 `AspectJExpressionPointcut` 객체들을 관리하고 있으며, `AspectJExpressionPointcut` 객체들은 Spring 빈 팩토리에 의해서 관리되고 있습니다.
- RequestMappingHandlerAdapter는 HTTP 요청을 처리하는 동안에 메모리를 사용하는데, 이 또한 메모리 사용의 일부로 일부로 보입니다.

#### 해결 방안
- **메모리 최적화**: AOP 관련 객체들이 지나치게 많은 메모리를 사용하고 있다면, AOP 사용 범위를 좁히거나 AspectJ 표현식을 최적화하여 메모리 사용을 줄일 수 있습니다.
- **GC(Garbage Collection)**: 주기적인 GC 수행을 통해서 메모리 할당 후 해제되지 않는 객체들을 관리하는 것도 필요할 수 있습니다.
- **메모리 프로파일링**: `jvisualvm`이나 `jprofiler`와 같은 도구를 사용하여 메모리 분석을 실시간으로 진행하고, 불필요한 객체들을 추적하여 메모리 누수를 방지할 수 있습니다.

#### 결론
AspectJExpressionPointcut 객체들이 많은 메모리를 차지하고 있습니다. Spring 애플리케이션에서 AOP 기능이 메모리 사용에 영향을 주고 있습니다. 이 문제를 해결하기 위해서는 AOP 구성이나 관련 빈을 최적화하고 메모리 프로파일링 기능을 통해서 추가적인 개선점을 찾는 것이 필요합니다.


### Problem Suspect 2
보고서 원문은 다음과 같습니다.
```
439 instances of **java.lang.ref.Finalizer**, loaded by **<system class loader>**occupy **9,319,336 (11.43%)** bytes. 

**Keywords**

- java.lang.ref.Finalizer
```

이 보고서는 `java.lang.ref.Finalizer` 객체들이 메모리에서 많은 공간을 차지하고 있다는 내용을 담고 있습니다. 이 객체들은 Java의 **Finalizer** 매커니즘과 관련이 있으며, Finalizer는 객체가 가비지 컬렉션될 때 실행되는 메서드를 정의하는 역할을 하고 있습니다.

#### 1. Finalizer 객체들에 대한 분석
- 439개의 `java.lang.ref.Finalizer` 인스턴스가 9,319,336 byte(11.43%, 8.9MB)의 메모리를 차지하고 있습니다. 즉, **Finalizer 인스턴스가 전체 메모리에서 11.43%를 차지하고 있습니다.**
- `Finalizer` 클래스는 Java의 finalize() 메서드를 구현하는데 사용됩니다. **finalize() 메서드는 객체가 더 이상 사용되지 않을 때 리소스를 정리하거나 cleanup 작업을 수행하도록 설계되어 있습니다.** 그러나 이 메커니즘은 메모리 효율성에 문제가 있을 수 있기 때문에, 현대 Java에서는 finalize() 메서드 사용을 지양하는 추세입니다.

#### 2. Finalizer의 역할
