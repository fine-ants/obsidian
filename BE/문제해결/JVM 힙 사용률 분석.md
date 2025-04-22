
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
- `java.lang.ref.Finalizer` 는 Finalizer Queue에 추가된 객체들을 추적하고, 그 객체가 가비지 컬렉션을 통해 제거 될 때 특정 작업을 실행하는데 사용됩니다. 예를 들어, finalize() 메서드가 정의되어 있으면, JVM은 이 메서드를 호출하여 리소스를 해제할 수 있습니다.
- 이 객체들은 Finalizer Queue에서 관리되며, 객체가 가비지 컬렉션되기 전에 `finalize()` 메서드가 호출됩니다. 그러나 **finalizer() 메서드는 성능 저하를 일으킬 수 있기 때문에, 자원 해제 로직은 `try-with-resources` 또는 명시적인 자원 관리 방법으로 대체하는 것이 좋습니다.**

#### 3. 메모리 사용 비율
- 439개의 `Finalizer` 인스턴스가 메모리의 11.43%를 차지하고 있다는 사실은, 메모리에서 상당한 크기를 차지하고 있음을 나타냅니다. 이는 `Finalizer` 인스턴스들이 Finalizer Queue에 등록되어 있는 상태에서 메모리 상에 남아 있음을 나타냅니다.
- 가비지 컬렉션이 제대로 이루어지지 않거나, finalize() 메서드가 충분하고 빠르게 처리되지 않기 때문일 수 있습니다.

#### 4. FInalizer 메커니즘에 대한 고려 사항
- **성능 문제** : `finalize()` 메서드는 예기치 않게 지연을 초래하거나 메모리 누수를 일으킬 수 있기 때문에, finalize() 사용을 최소화하는 것이 좋습니다.
- **메모리 관리 문제**: `Finalizer` 객체들이 많이 사용되는 경우, Finalizer Queue에 등록된 객체들이 제거되지 않고, 메모리에 남을 수 있어서 메모리 누수가 발생할 위험이 있습니다.
- **가비지 컬렉터의 성능 저하**: `finalize()` 메서드가 호출되려면, 추가적인 가비지 컬렉션 작업이 필요하기 때문에 이를 통해 애플리케이션 성능이 저하될 수 있습니다.

#### 5. 대처 방법
- `AutoClosable` 인터페이스와 `try-with-resources`
	- `AutoClosable` 인터페이스를 구현한 객체는 `try-with-resources` 구문을 사용하여 자동으로 리소스를 정리할 수 있습니다. 이는 `finalize()` 메서드보다 더 안전하고 효율적인 방법입니다.
- 명시적 자원 해제
	- 객체가 더이상 필요하지 않으면, 명시적으로 자원을 해제하는 방법이 권장됩니다. 예를 들어, `close()` 메서드를 호출하여 연결을 종료하거나 파일을 닫는 방식입니다.

#### 6. 최적화 방법
- **Finalizer 객체의 사용 감소**: `finalize()` 메서드나 `java.lang.ref.Finalizer` 사용을 최소화하여 메모리 사용을 줄입니다. `finalize()` 메서드 대신에 `try-with-resources` 구문을 사용하는 것이 좋습니다.
- **메모리 누수 점검**: `Finalizer` 메커니즘에 의해 메모리 누수가 발생할 수 있으므로, 이를 점검하고 가능한 한 다른 방법으로 리소스를 해제하는 로직을 작성해야 합니다.
- **가비지 컬렉션 모니터링**: `finalize()` 메서드가 호출되기 전에 가비지 컬렉션이 적절히 실행되고 있는지 확인하고, **Full GC**가 정상적으로 수행되도록 시스템을 모니터링해야 합니다.

#### 결론
- 439개의 java.lang.ref.Finalizer 인스턴스들이 메모리의 11.43%를 차지하고 있습니다. finalize() 메서드에 의해서 관리되는 객체들이 메모리를 많이 차지하고 있다는 의미입니다. 이 문제는 `finalize()` 메커니즘으로 인한 성능 저하와 메모리 누수를 나타낼 수 있습니다.
- 이러한 상황을 개선하기 위해서는 `finalize()` 메서드 대신에 AutoClosable 인터페이스나 명시적인 자원 해제 방법을 사용하고 Finalizer 객체의 수를 줄이거나 제거하는 것이 좋습니다.


pointcutDeclarationScope