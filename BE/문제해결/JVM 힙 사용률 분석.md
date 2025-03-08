
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

#### 