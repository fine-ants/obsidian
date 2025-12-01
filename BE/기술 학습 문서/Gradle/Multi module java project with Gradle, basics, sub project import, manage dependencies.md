
- [[#개요|개요]]
- [[#멀티 모듈 프로젝트는 무엇인가|멀티 모듈 프로젝트는 무엇인가]]
- [[#루트 프로젝트 생성 및 모듈 생성|루트 프로젝트 생성 및 모듈 생성]]
- [[#common 모듈 생성하기|common 모듈 생성하기]]
- [[#모든 의존성들을 외부의 파일에 저장하는 방법|모든 의존성들을 외부의 파일에 저장하는 방법]]
- [[#정리|정리]]



## 개요
- Gradle 기반으로 멀티 모듈 Project를 생성하는 방법을 학습
- project, subprojects, allprojects을 통하여 특정 모듈들에 설정하는 방법을 학습
- 모듈간에 다른 의존성을 공유하는 방법을 학습
- 외부 파일을 기반으로 의존성 라이브러리를 관리하는 방법을 학습

## 멀티 모듈 프로젝트는 무엇인가
**멀티 모듈 프로젝트는 여러개의 작은 프로젝트들로 구성된 프로젝트입니다.** 해당 프로젝트에는 루트 프로젝트가 존재하고 루트 프로젝트 아래에 여러개의 모듈이 존재할 수 있습니다. 
다음 프로젝트를 보면 프로젝트 이름은 "gradlebasics" 라는 이름이 프로젝트 이름이고 하나의 gradlebasics 라는 프로젝트 이름 아래에 sub-project-1, sub-project-2, common라는 모듈이 포함되어 있는 형태입니다. 물론 gradlebasics 라는 프로젝트 자체가 하나의 모듈로서 작동할 수도 있습니다.
![](refImg/Pasted%20image%2020251129152422.png)


## 루트 프로젝트 생성 및 모듈 생성
### gradlebasics 루트 프로젝트 생성
gradlebasics라는 이름의 루트 프로젝트를 생성합니다.
![](refImg/Pasted%20image%2020251129153119.png)

gradlebasics 프로젝트 생성 확인
```shell
./gradlew build
```
![](refImg/Pasted%20image%2020251129153217.png)

다음 화면을 보면 gradlebasics라는 프로젝트가 루트 프로젝트가 되는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251129153226.png)

### sub-project-1 모듈 생성
sub-project-1 모듈 생성시 "Module..." 메뉴를 통하여 생성하는 것이 아닌 단순히 Directory 메뉴를 통하여 생성해봅니다.
![](refImg/Pasted%20image%2020251129153352.png)

생성한 sub-project-1 디렉토리가 모듈로 취급하기 위해서 루트 프로젝트의 **`settings.gradle`** 파일을 수정하여 sub-project-1 모듈을 포함하도록 합니다.
![](refImg/Pasted%20image%2020251129153526.png)

모듈 프로젝트 확인
```shell
./gradlew projects
```

다음 실행 화면을 보면 루트 프로젝트가 "gradlebasics"이고 모듈 프로젝트가 "sub-project-1" 프로젝트인 것을 볼수 있습니다. 우리는 sub-project-1 프로젝트를 하나의 모듈로 취급합니다.
![](refImg/Pasted%20image%2020251129153705.png)

sub-project-1 모듈에 소스 코드 디렉토리 생성
![](refImg/Pasted%20image%2020251129153946.png)

sub-project-1 모듈, 패키지 생성
![](refImg/Pasted%20image%2020251129154459.png)

subprojectone 패키지에 클래스 생성
![](refImg/Pasted%20image%2020251129154629.png)


`build.gradle` 파일 생성
해당 디렉토리에 `build.gradle` 파일을 생성한다는 의미는 해당 디렉토리를 독립적인 Gradle 모듈로 정의하고 그 모듈의 **빌드 방식과 설정을 명시**하겠다는 의미입니다.
![](refImg/Pasted%20image%2020251129154711.png)


> [!NOTE] IntelliJ IDEA에서 모듈 취급하기
> 서브 모듈을 디렉토리와 같이 직접적으로 생성하는 경우 IntelliJ IDEA 편집창에서 해당 디렉토리를 서브 모듈로 취급하지 않을 수 있습니다. 이 문제를 해결하기 위해서 IntelliJ의 Gradle 메뉴에서 **"Reload All Gradle Project"** 기능을 실행하여 sub-project-1와 같은 디렉토리를 모듈 취급하게 할 수 있습니다.
![](refImg/Pasted%20image%2020251201125226.png) ![](refImg/Pasted%20image%2020251201125253.png)


### sub-project-2 모듈 생성
sub-project-1 모듈과 같이 디렉토리를 직접 생성하는 방법이 아닌 Intellij Module 메뉴를 통해서 생성해봅니다.
![](refImg/Pasted%20image%2020251129155343.png)

생성창에서 이름과 JDk 버전, 상위 모듈 또는 프로젝트를 설정할 수 있습니다.
![](refImg/Pasted%20image%2020251129155455.png)

sub-project-2 모듈 생성 확인
![](refImg/Pasted%20image%2020251129155534.png)

`build.gradle` 확인
파일을 확인하면 sub-project-2 모듈의 독립적인 설정들이 존재하는데, 이번 학습에서는 중복적인 설정을 모두 루트 프로젝트의 Gradle 설정에서 제어할 예정이기 때문에 다음 설정들을 모두 제거합니다. 이는 sub-project-1 모듈 또한 동일합니다.
![](refImg/Pasted%20image%2020251129160512.png)

![](refImg/Pasted%20image%2020251129160613.png)
![](refImg/Pasted%20image%2020251129160708.png)

## 루트 프로젝트 - project 설정
`build.gradle` 파일에 project 설정을 이용하여 특정 모듈의 Gradle 설정을 추가할 수 있습니다.

project 설정을 활용하여 서브 모듈인 sub-project-1 모듈에 "hello" 라는 이름의 테스크를 추가해보겠습니다.
추가적으로 sub-project-1 모듈에 java 플러그인을 추가합니다. java 플러그인을 추가함으로써 sub-project-1 모듈은 표준 자바 프로젝트로 인식하고, 자바 코드를 빌드하는데 필요한 모든 설정(compileJava, jar, test 등)을 자동으로 추가합니다.
![](refImg/Pasted%20image%2020251129162906.png)


sub-project-1 hello Task 수행
```shell
./gradlew :sub-project-1:hello
```

실행 결과를 보면 정상적으로 hello 테스크를 수행하여 프로젝트 이름을 출력한 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251129163027.png)

### 루트 프로젝트 - subprojects 설정
project 설정을 이용하여 특정 모듈에 테스크 및 플러그인을 적용할 수 있었습니다. 하지만 만약에 동일한 테스크 및 플러그인 설정을 다른 모듈에도 적용하기 위해서는 어떻게 해야할까요? project 설정을 추가하여 동일한 코드의 테스크와 플러그인 설정을 추가해야 할 것입니다. 하지만 이러한 방법은 코드 중복이 발생합니다.

위와 같은 중복 문제를 해결하기 위해서 subprojects 설정을 사용하여 해결할 수 있습니다. **subprojects 설정을 사용하면 루트 프로젝트를 제외한 서브 모듈들에 동일한 설정을 제공**할 수 있습니다.

subprojects 설정을 사용하여 sub-project-1, sub-project-2 모듈에 java 플러그인, spring boot 플러그인, hello 테스크를 적용해보겠습니다.
![](refImg/Pasted%20image%2020251129163933.png)


루트 프로젝트 hello 테스크 테스트
```shell
./gradlew hello
```

실행 결과를 보면 2개의 서브모듈의 hello 테스크가 실행된 것을 확인할 수 있습니다.
![](refImg/Pasted%20image%2020251129164132.png)


### 루트 프로젝트 - allprojects 설정
우리는 subprojects 설정을 이용하여 루트 프로젝트를 제외한 서브 모듈들에 동일한 설정을 적용할 수 있었습니다. 하지만 만약에 루트 프로젝트를 포함한 서브 모듈들에도 동일한 설정을 적용하기 위해서는 어떻게 해야 할까요? 
위 문제를 해결하기 위해서 allprojects 설정을 사용할 수 있습니다. allprojects 설정을 사용하면 루트 프로젝트 포함 서브 모듈들에도 동일한 설정을 적용할 수 있습니다.

루트 프로젝트의 build.gradle 파일에 다음과 같이 allprojects 설정을 사용하여 동일한 설정을 적용합니다.
![](refImg/Pasted%20image%2020251201134954.png)
 
subprojects 설정에서 등록했었던 hello 테스크를 allprojects 설정으로 이동시킵니다. 이렇게 함으로써 루트 프로젝트 자체의 hello 테스크를 수행할 수 있습니다.
![](refImg/Pasted%20image%2020251129165345.png)

루트 프로젝트의 hello 테스크 실행
```shell
./gradlew hello
```

실행 결과를 보면 루트 프로젝트의 hello 실행만이 아닌 서브 모듈들의 hello도 실행한 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251129165522.png)


> [!NOTE] java toolchain 설정을 이용하여 JDK 버전 명시
> java toolchain 설정을 이용하여 실행하는 Gradle의 JDK 버전을 명세적으로 설정할 수 있습니다.
> ![](refImg/Pasted%20image%2020251201135103.png)
> 


## common 모듈 생성하기
서브 모듈들간에 공통적으로 사용하는 의존성 설정이 존재한다면 common과 같은 이름의 모듈을 생성하여 sub-project-1, sub-project-2과 같은 서브 모듈들에서 설정을 의존할 수 있습니다.

common 모듈 생성
![](refImg/Pasted%20image%2020251130122407.png)

패키지 생성
![](refImg/Pasted%20image%2020251130122835.png)

CommonEntity 클래스 생성
```java
@AllArgsConstructor  
public class CommonEntity {  
    private final String id;  
}
```

build.gradle 설정 추가
common 모듈을 별도의 jar 파일로 생성할 수 있도록 다음 설정을 추가합니다.
```gradle
jar.enabled = true
```
- 해당 설정을 true로 설정하면 모듈을 jar 파일로 생성할 수 있습니다.
- 이 설정을 하는 이유는 프로젝트를 다른 곳에서 라이브러리로 사용해야할 때 사용합니다.

## 모듈간 의존성 추가하기
프로젝트 내의 서브 모듈이 다른 모듈의 클래스를 참조하기 위해서는 모듈간의 의존성을 추가하여 참조할 수 있습니다. 예를 들어 sub-project-1 모듈이 common 모듈의 CommonEntity 클래스를 참조하기 위해서 모듈간 의존성을 추가해보도록 하겠습니다.

sub-project-1 모듈에서 common 모듈에 대한 모듈간 의존성 추가, build.gradle
```
dependencies {  
    implementation project(':common')  
}
```

sub-project-1, ModuleOneUser 클래스에서 common 모듈에 있는 CommonEntity 클래스를 참조합니다.
다음 코드를 보면 import문을 이용해서 common 모듈에 있는 CommonEntity를 가져오는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251201102757.png)

**모듈간에 의존성들을 추가 방법 정리**
다음 그림을 보면 moudle-2 모듈에서는 moudle-1을 의존하도록 설정되어 있습니다. 이렇게 함으로써 moudle-2의 프로덕션 코드에서 module-1 모듈에 존재하는 코드를 import하여 가져올 수 있습니다.
단, moudle-1 모듈에 존재하는 'commons-io:commons-io:2.8.0' 의존성은 module-2에서 import하여 가져올수 없습니다. 이는 해당 의존성 설정이 **implementation**으로 설정되어 있기 때문에 전이 의존성이 발생하지 않습니다.
하지만 api 의존성으로 설정되어 있는 'junit:junit:4.2' 의존성 같은 경우에는 api로 설정되어 있기 때문에 전이 의존성이 발생하여 module-2 모듈에서 참조가 가능합니다.
![](refImg/Pasted%20image%2020251201103402.png)
- moudle-2 모듈에서는 commons-io 라이브러리에 접근 불가능
- 단, module-2 모듈에서는 junit 라이브러리는 접근 가능


### common 모듈에 guava 의존성을 추가하기
common 모듈에 guava 의존성을 추가하여 common 모듈을 의존하고 있는 sub-project-1 모듈에서도 guava 라이브러리 클래스를 사용할 수 있도록 합니다.

루트 프로젝트에 있는 guava 의존성 설정을 common 모듈로 이동시킵니다.
![](refImg/Pasted%20image%2020251201105609.png)

common 모듈에서 guava 라이브러리의 클래스를 사용하여 객체를 생성해봅니다. 다음 코드를 보면 common 모듈의 CommonEntity 클래스에서 guava 라이브러리 클래스인 ImmutableMap 클래스를 참조하는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251201105725.png)

그러면 이번에는 common 모듈을 참조하고 있는 sub-project-1 모듈을 대상으로 guava 라이브러리 클래스인 ImmutableMap 객체를 생성할 수 있는지 확인해봅니다. 다음 화면을 보면 sub-project-1 모듈에서 ImmutableMap 클래스를 참조할 수 없는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251201142127.png)

sub-project-1 모듈에서 common 모듈의 guava 의존성을 가져오지 못하는 이유는 **guava 의존성이 implementation 의존성으로 설정되었기 때문**입니다. implementation 의존성으로 설정되면 **전이 의존성**이 발생하지 않아서 해당 모듈을 참조하는 다른 모듈에서 implementation 으로 설정한 의존성 라이브러리를 참조할 수 없습니다.
![](refImg/Pasted%20image%2020251201111126.png)

위와 같은 문제를 해결하기 위해서는 implementation 의존성 설정이 아닌 **api 의존성 설정**으로 해야 합니다.
- api 의존성 설정을 사용하기 위해서는 java-library 플러그인을 추가해야 합니다.
- 해당 실습에서는 루트 프로젝트의 subprojects 설정에 java-library 플러그인을 적용한 상태입니다.
![](refImg/Pasted%20image%2020251201111420.png)

api 의존성 설정으로 변경후 Gradle을 리로드한 다음에 sub-project-1 모듈의 ModuleOneUser 클래스를 보면 ImmutableMap 클래스를 참조할 수 있습니다.
![](refImg/Pasted%20image%2020251201142436.png)

이번에는 Gradle Build하여 빌드가 되는지 테스트해봅니다.
![](refImg/Pasted%20image%2020251201111518.png)


## 외부 파일을 이용한 의존성 관리하는 방법
다음 의존성 설정은 루트 프로젝트의 의존성 설정입니다. 그중에서 lombok 라이브러리(1.18.22)를 compileOnly, annotationProcessor 의존성 설정한 것을 볼수 있습니다. 
![](refImg/Pasted%20image%2020251201143118.png)

루트 프로젝트에 `dependencies.gradle` 파일 생성
![](refImg/Pasted%20image%2020251201112556.png)

루트 프로젝트에 있는 모든 의존성을 dependencis.gradle 파일로 이동시키고 ext 설정을 이용해서 다음과 같이 설정합니다. 또한 common 모듈에 있는 guava 의존성 또한 가져와 설정하도록 합니다.
![](refImg/Pasted%20image%2020251201113101.png)

루트 프로젝트의 build.gradle에서 `dependencis.gradle` 파일을 참조하여 의존성 설정을 적용합니다.
![](refImg/Pasted%20image%2020251201115912.png)
![](refImg/Pasted%20image%2020251201114358.png)

위와 같이 설정한 다음에 build 하여 설정이 적용되는지 확인해봅니다. 실행 결과를 보면 정상적으로 빌드된 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251201114638.png)

sub-project-1 모듈에서 junitJupiterApi가 정상적으로 동작하는지 테스트 클래스를 생성하고 실행해봅니다.
![](refImg/Pasted%20image%2020251201120005.png)

실행 결과를 보면 정상적으로 테스트가 성공한 것을 볼수 있습니다.
```shell
./gradlew test
```
![](refImg/Pasted%20image%2020251201120020.png)

## 정리
- 멀티 모듈 Gradle 프로젝트를 생성하는 방법을 학습함
- 여러 모듈(프로젝트)에서 의존성을 관리하는 방법을 학습함
