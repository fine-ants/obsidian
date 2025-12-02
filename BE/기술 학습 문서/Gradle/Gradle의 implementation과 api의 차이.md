

## 1. Gradle Java 플러그인과 Java-Library 플러그인
### Java 플러그인은 무엇인가?
Gradle 설정에서 Java 플러그인을 추가하면 해당 프로젝트를 자바 프로젝트로 만들고 자바 소스코드를 컴파일, 테스트, 빌드하는데 필용한 기능들을 Gradle에 부여해줍니다.

```gradle
plugins {  
    id 'java'
}
```

Java 플러그인을 추가하면 다음 그림과 같은 테스크들을 사용할 수 있습니다. 각 테스크들간에 화살표 관계는 해당 테스크를 완료하기 위한 선행 조건 테스크들을 의미합니다. 예를 들어 build 테스크를 수행하기 위해서는 check, assemble 테스크가 먼저 수행 및 완료되어야 합니다.
![그림](refImg/Pasted%20image%2020251201155125.png)

자바 컴파일러가 자바 소스 코드(`*.java`)를 컴파일 하게 되면 바이트 코드(`*.class`)를 생성합니다. 이러한 바이트 코드들은 JVM(Java Virtual Machine)에 의해서 읽어지게 됩니다. 그리고 읽어진 바이트 코드들은 기계어로 번역됩니다. JVM이 자바 소스코드를 컴파일하거나 컴파일된 바이트코드를 읽어서 실행하기 위해서 파일과 패키지를 탐색하는데, 이러한 경로를 **클래스패스(classpath)**라고 합니다.

클래스패스에는 두가지 종류가 존재합니다. 첫번째는 컴파일 클래스 패스(compile class path), 두번째는 런타임 클래스 패스(runtime class path)입니다.
- 컴파일 클래스 패스(compile classpath) : 자바 소스 코드(`*.java`)를 바이트 코드(`*.class`)로 컴파일 할때 탐색하는 경로
- 런타임 클래스 패스(runtime classpath) : 컴파일 된 바이트 코드(`*.class`)을 JVM이 읽기 위해서 탐색하는 경로

**Gradle의 클래스패스 의존성 설정**
Gradle을 기반으로 자바 애플리케이션을 빌드하기 위해서 설정하는 의존성 설정이 존재합니다. 그래서 Gradle 설정에서 각각의 라이브러리의 의존성을 추가할때 어느 범위로 노출시킬 것인지 결정할 수 있습니다. 종류는 다음과 같습니다.
- compileOnly : 컴파일 클래스 경로에만 라이브러리를 설정함
- runtimeOnly : 런타임 클래스 경로에만 라이브러리를 설정함 
- implementation : 컴파일 클래스 경로, 런타임 클래스 경로 두곳에 라이브러리를 설정함
- api : 컴파일 클래스 경로, 런타임 클래스 경로 두곳에 라이브러리를 설정함

### Java-Library 플러그인은 무엇인가?
Java-Library 플러그인은 기본 Java 플러그인의 모든 기능을 포함하고, 라이브러리 개발에 필요한 특화된 규칙(예: 프로덕션 코드는 src/main/java에 위치)을 추가합니다.
Java 플러그인과의 차이점은 라이브러리 개발시 어떤 의존성을 라이브러리의 외부 API로 노출하고(다른 프로젝트가 접근 가능), 어떤 의존성을 내부 구현으로 숨길지(다른 프로젝트가 접근 불가)을 정의할 수 있습니다.(이때 `api`, `implementation` 설정이 사용됨)
Java-Library를 사용하면 java 플러그인이 제공하는 기본적인 디렉토리 구조, 빌드 테스크(compileJava, jar 등), 그리고 의존성 설정(testImplementation 등)을 모두 자동으로 사용할 수 있습니다.


## 2. Gradle implementation과 api의 차이
### 전이 의존성이란 무엇인가?
Gradle 설정에서 의존성 라이브러리 설정시 다양한 설정이 옵션이 존재합니다. 대표적으로 implementation과 api 의존성 설정이 존재합니다. 두 설정은 모두 컴파일 클래스경로(compile classpath)와 런타임 클래스경로(runtime classpath) 두곳 모두에 라이브러리를 설정하는 공통점을 가지고 있습니다. 하지만 대표적인 차이점은 **전의 의존성(transitive dependency)의 컴파일 경로 노출 여부**가 있습니다. 
전이 의존성은 어떤 프로젝트 A가 직접적으로 명시하지 않은 라이브러리 C인데도 불구하고, 프로젝트 A가 다른 프로젝트 B를 의존하는 것만으로도 라이브러리 C를 사용할 수 있다면 전이 의존성이 있는 것입니다.
![그림](refImg/Pasted%20image%2020251202115140.png)

### implementation과 api의 차이
**`api` 의존성 설정은 전이 의존성을 허용하고 있습니다.** 예를 들어 프로젝트 A,B,C가 존재하고 프로젝트 B는 프로젝트 C를 의존하고 프로젝트 A는 프로젝트 B를 의존하는 관계라고 가정합니다. 그림으로 보면 다음과 같습니다.
![그림](refImg/Pasted%20image%2020251202120803.png)

위 그림을 보면 프로젝트 C에는 Hello라는 클래스가 public으로 구현되어 있습니다. 그런한 프로젝트 C를 프로젝트 B는 api 설정으로 의존성 설정되어 있습니다. `api`로 설정하였기 때문에 전이 의존성을 허용합니다. 그래서 프로젝트 B를 의존하고 있는 프로젝트 A에서는 프로젝트 C에 정의되어 있는 Hello 클래스를 참조하여 객체를 생성할 수 있습니다.

반면에 `implementation` 의존성 설정은 전이 의존성을 허용하고 있지 않습니다. 예를 들어 위 그림에서 프로젝트 B가 프로젝트 C의 의존성 설정을 `api`가 아닌 `implementation`으로 설정해보겠습니다. 그렇게 되면 상황은 다음 그림과 같이 됩니다.
프로젝트 B의 의존성 라이브러리 설정에서 `implementation`으로 설정된 의존성 라이브러리들은 내부 구현으로 숨겨져 있기 때문에 프로젝트 A가 프로젝트 C의 Hello 클래스를 사용할 수 없습니다.
![그림](refImg/Pasted%20image%2020251202122742.png)

### implementation과 api의 사용
`api` 대신 `implementation`을 사용하는 경우 장점은 다음과 같습니다.
- 컴파일 클래스패스(compile classpath)에 `implementation`으로 설정된 의존성 라이브러리들이 노출되지 않아서 해당 라이브러리들에 대해서 종속적이지 않게 됩니다.
- 의존성 라이브러리들이 노출되지 않아서 그만큼 컴파일 속도가 빨라집니다.
- `implementation`으로 설정된 의존성 라이브러리들이 변경되면 다시 컴파일하지 않아도 되서 재컴파일 횟수가 줄어듭니다.

`api`는 애플리케이션 바이너리 인터페이스(Application Binary Interface)로써 특정 상황에서 사용하면 좋습니다. 애플리케이션 바이너리 인터페이스란 컴파일된 코드(Binary) 레벨에서 다른 모듈과의 호환성을 정의하는 인터페이스입니다.
프로젝트에서 ABI에 해당하는 타입이 변경되면, 해당 프로젝트를 의존하는 외부의 프로젝트는 반드시 재컴파일해야 합니다. 반면에 ABI에 해당하지 않는 타입들은 내부 구현만 변경되면 외부의 프로젝트들은 다시 컴파일할 필요없이 빌드 속도가 빨라집니다.

`api`(ABI에 노출되는 부분)
`api`로 설정된 의존성 라이브러리들은 프로젝트를 사용하는 외부의 프로젝트들로부터 노출됩니다.

`api`를 사용해야 하는 경우(ABI 해당)
다음의 경우는 외부 프로젝트들이 해당 타입을 직접 알아야 하거나 접근해야 하는 경우입니다.

1. 부모 클래스 또는 인터페이스에 사용되는 타입
	- 예를 들어 당신의 프로젝트 B가 **외부 라이브러리 A의 인터페이스와 상속받은 클래스를 외부에 공개할 경우**에 외부의 프로젝트인 C는 외부 라이브러리 A의 존재를 알고 있어야 합니다.
![그림](refImg/Pasted%20image%2020251202131147.png)

2. public 메서드 파라미터/반환 타입으로 사용하는 타입
	- 예를 들어 `public List<User> getUsers()` 메서드와 같이 외부 라이브러리 A의 `List`나 `User` 타입을 반환하면, 당신의 프로젝트를 사용하는 외부의 프로젝트들은 컴파일 시점에 외부 라이브러리 A의 `User` 타입을 알고 있어야 합니다.
![그림](refImg/Pasted%20image%2020251202131252.png)

3. public 필드에 사용되는 타입
	- 외부에 공개된 타입이나 annotation 타입은 외부에서 직접 참조하므로 ABI에 해당됩니다.
	- 예를 들어 MyLibrary 모듈(프로젝트)이 구현한 클래스에서 외부 라이브러이인 Gson 라이브러리의 JsonElement 타입을 public으로 설정하여 구현하였습니다. 이러한 MyLibrary 모듈을 의존하는 외부 프로젝트인 AppModule이 DataContainer 객체의 element public 필드를 참조한다면 AppModule 모듈은 Gson 라이브러리의 JsonElement를 import 할 수 있어야 합니다. 이렇게 하기 위해서 gson 라이브러리를 api로 의존성 해야 합니다.
![그림](refImg/Pasted%20image%2020251202133809.png)

4. public annotation 타입
	- 외부에 공개되는 annotation 타입은 외부에서 직접 참조할 수 있으므로 ABI에 해당됩니다.
	- 예를 들어 MyLibrary 모듈에서 validation 외부 라이브러리를 의존하고 그 라이브러리의 애노테이션 중 하나인 `@NotNull`을 필드에 사용합니다. 그리고 MyLibrary를 의존하는 외부 프로젝트인 AppModule 모듈이 프로덕션 코드 중에서 매개변수로 받은 객체의 클래스 정보 중에서 NotNull을 가지고 있는지 확인합니다. 이 상황에서 AppMoudle은 validation 외부 라이브러리를 알아야 합니다.
![그림](refImg/Pasted%20image%2020251202140232.png)

`api`를 사용하지 않아야 하는 경우 (ABI에 해당 안됨)
1. 메서드의 바디에서만 사용되는 타입
	- 예를 들어 MyLibrary 모듈은 Guava 라이브러리를 의존하고 있습니다. 그리고 processData 메서드를 구현하는 과정에서 Guava 라이브러리의 ImmutableList를 import하여 사용하고 있습니다. 하지만 반환 타입은 List 타입을 사용하고 있기 때문에 외부 프로젝트에서는 ImmutableList 타입을 알지 않아도 됩니다. 그래서 외부 프로젝트인 AppMoudle에서는 DataProcessor 객체를 사용할때 processData 메서드를 호출하여도 ImmutableList 타입은 외부에 노출되지 않고 사용할 수 있습니다. 이러한 경우에 일부로 guava 라이브러리를 `api`로 노출시키지 않아도 됩니다.
![그림](refImg/Pasted%20image%2020251202142134.png)

2. private 멤버로 사용되는 타입
	- 내부에서만 사용되는 private 멤버로 사용되는 타입이라면 `api` 사용하지 않아도 됩니다.
	- 예를 들어 MyLibrary 모듈에서 lombok 라이브러리를 의존합니다. 그리고 Hello 클래스 구현시 Lombok 라이브러리의 Logger 클래스를 사용할때 private로 선언되어 있고 Hello 클래스에서만 내부적으로 사용되고 있습니다. 이러한 경우에 외부 프로젝트인 AppModule에서 MyLibrary 모듈을 의존하고 Hello 객체를 생성하고 사용해도 Logger에 대한 노출이 없기 때문에 Lombok이 변경되어도 재컴파일하지 않아도 됩니다.
![그림](refImg/Pasted%20image%2020251202145159.png)

3. 내부 클래스에서 발견되는 타입
	- 외부 라이브러리가 오직 외부에 공개되지 않은 내부 클래스(Inner Class)나 익명 클래스에서만 사용될때 해당됩니다.
	- 예를 들어 MyLibrary 모듈이 StringUtils 외부 라이브러리를 의존합니다. 그러나 MyLibrary 모듈을 사용하는 외부의 프로젝트에서는 ObjectFactory 객체가 create 메서ㄷ를 수행할때 StringUtils 라이브러리를 참조할 필요가 없습니다.
![그림](refImg/Pasted%20image%2020251202150247.png)


### implementation과 api 차이 요약
| **구분**                    | **implementation (권장)**                        | **api (제한적 사용)**                                          |
| ------------------------- | ---------------------------------------------- | --------------------------------------------------------- |
| **의존성 전파 (Transitivity)** | **차단** (노출 안 함)                                | **허용** (노출 함)                                             |
| **외부 노출**                 | 이 모듈을 사용하는 **외부 모듈에게 라이브러리 노출 안 함** (전이 의존성 X) | 이 모듈을 사용하는 **외부 모듈에게 라이브러리 노출 함** (전이 의존성 O)              |
| **사용 목적**                 | 모듈의 **내부 구현**에만 사용하는 의존성 (예: 유틸리티, 로깅 구현체)     | 모듈의 **공개된 API**를 구성하는 데 사용되는 의존성 (예: Public 메서드의 파라미터 타입) |
| **빌드 성능**                 | **우수함.** 내부 구현 변경 시 외부 모듈은 재컴파일 불필요.           | **낮음.** `api` 의존성 변경 시 이를 사용하는 모든 외부 모듈이 **재컴파일 필요.**     |

## References
- https://mangkyu.tistory.com/296
- https://docs.gradle.org/current/userguide/java_library_plugin.html
- https://docs.gradle.org/current/userguide/java_plugin.html
- https://robin00q.tistory.com/15




