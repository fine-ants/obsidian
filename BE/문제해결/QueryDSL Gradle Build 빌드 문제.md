
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