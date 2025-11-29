
## 배경
IntelliJ IDEA가 아닌 Gradle 기반으로 빌드 명령어를 수행 시 QueryDSL의 QClass를 찾을 수 없다고 에러를 출력하였습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129134704.png)

하지만 IntelliJ IDEA를 이용하여 빌드를 수행할 때는 정상적으로 QueryDSL의 QClass를 생성하고 빌드 완료된 것을 볼수 있습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129134821.png)

Project Version 정보
- Spring Boot 3.1.1
- Java : temurin-17(open jdk 17)
- QueryDSL : 5.0.0
- Lombok : 1.18.22

Gradle QueryDSL Task 설정
![](BE/문제해결/refImg/Pasted%20image%2020251129135248.png)

## 원인
IntelliJ IDE를 이용하여 빌드를 수행할 때는 temurin-17(open jdk17) 버전으로 빌드를 수행했기 때문에 정상적으로 완료된 것입니다. 다음 화면을 보면 Project의 SDK가 temurin-17로 설정된 것을 볼수 있습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129135934.png)

하지만 터미널에서 Gradle 명령어를 이용하여 build를 수행하는 경우에는 다른 JDK 버전으로 수행된 것을 볼수 있었습니다.
```shell
./gradlew build --info
```

다음 실행 결과를 보면 Gradle build 명령어 실행시 정작 실행되는 JDK 버전은 temurin-21(open jdk21)인것을 볼수 있습니다. 제 로컬 개발 환경 경우에는 mac os 운영체제에 temurin-17과 temurin-21이 같이 설치되어 있는 상태입니다. 
![](BE/문제해결/refImg/Pasted%20image%2020251129140125.png)

**open jdk21 버전으로 컴파일 할때의 문제점은 Lombok 라이브러리와 버전 충돌이 발생한다는 점입니다.** 현재 Lombok 라이브러리 버전은 1.18.22 버전으로 JDK17 버전에 맞추어져 있는 최소 버전입니다. jdk21으로 컴파일시 Lombok 1.18.22 버전을 사용하면 컴파일러의 내부 API 충돌이 발생합니다.

다음 화면을 보면 1.18.30 버전부터 JDK21이 지원된다는 것을 알 수 있습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129140815.png)

반면에 1.18.22 버전부터 JDK17이 지원됩니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129140908.png)

정리하면 Gradle build 명령어 수행시 컴파일에 실행한 이유는 QueryDSL 문제 이전에 Gradle이 JDK21 버전으로 수행되고 Lombok 라이브러리와 버전 충돌이 발생하면서 Annotation Processor 작업이 실패한 것입니다. 그로인해서 QClass가 생성되지 않고 에러 결과에서는 QClass가 존재하지 않는다고 실패한 것입니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129141201.png)

## 해결 방법
위와 같은 컴파일 문제를 해결하기 위해서는 Gradle이 실행하는 JDK 버전을 open JDK17 버전으로 변경합니다.

프로젝트 루트 경로에 `gradle.properties` 파일을 추가합니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129141333.png)

gradle.properties 파일 수정
- `org.gradle.java.home` 프로퍼티에 사용하고자 하는 JDK 경로를 설정합니다.

```properties
org.gradle.java.home=/Library/Java/JavaVirtualMachines/temurin-17.jdk/Contents/Home
```

Gradle Build 테스트
```shell
./gradlew clean build --info
```

Gradle 로그를 보면 JDK17을 사용하는 것을 확인할 수 있습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129141629.png)


