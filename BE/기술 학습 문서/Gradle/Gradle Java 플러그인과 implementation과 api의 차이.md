

## 1. Gradle Java 플러그인과 Java-Library 플러그인
### Java 플러그인은 무엇인가?
Gradle 설정에서 Java 플러그인을 추가하면 해당 프로젝트를 자바 프로젝트로 만들고 자바 소스코드를 컴파일, 테스트, 빌드하는데 필용한 기능들을 Gradle에 부여해줍니다.

```gradle
plugins {  
    id 'java'
}
```

Java 플러그인을 추가하면 다음 그림과 같은 테스크들을 사용할 수 있습니다. 각 테스크들간에 화살표 관계는 해당 테스크를 완료하기 위한 선행 조건 테스크들을 의미합니다. 예를 들어 build 테스크를 수행하기 위해서는 check, assemble 테스크가 먼저 수행 및 완료되어야 합니다.
![](refImg/Pasted%20image%2020251201155125.png)

자바 컴파일러가 자바 소스 코드(`*.java`)를 컴파일 하게 되면 바이트 코드(`*.class`)를 생성합니다. 이러한 바이트 코드들은 JVM(Java Virtual Machine)에 의해서 읽어지게 됩니다. 그리고 읽어진 바이트 코드들은 기계어로 번역됩니다. JVM이 자바 소스코드를 컴파일하거나 컴파일된 바이트코드를 읽어서 실행하기 위해서 

클래스패스에는 두가지 종류가 존재합니다. 첫번째는 컴파일 클래스 패스(compile class path), 두번째는 런타임 클래스 패스(runtime class path)입니다.
- 컴파일 클래스 패스(compile classpath) : 자바 소스 코드(`*.java`)를 바이트 코드(`*.class`)로 컴파일 할때 탐색하는 경로
- 런타임 클래스 패스(runtime classpath) : 