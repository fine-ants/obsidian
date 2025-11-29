
## 배경
IntelliJ IDEA가 아닌 Gradle 기반으로 빌드 명령어를 수행 시 QueryDSL의 QClass를 찾을 수 없다고 에러를 출력하였습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129134704.png)

하지만 IntelliJ IDEA를 이용하여 빌드를 수행할 때는 정상적으로 QueryDSL의 QClass를 생성하고 빌드 완료된 것을 볼수 있습니다.
![](BE/문제해결/refImg/Pasted%20image%2020251129134821.png)

Project Version 정보
- Spring Boot 3.1.1
- Java : temurin-17(open jdk 17)
- com.querydsl:query-dsl-jpa:5.0.0:jakarta
- com.querydsl:querydsl-apt:5.0.0:jakarta
- 
