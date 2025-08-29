해당 문서는 Spring 프로젝트에서 Spring Rest Docs를 적용하기 위한 메뉴얼입니다.

### 1. asciidoctor 플러그인 및 Rest Docs 의존성 추가
spring 프로젝트의 build.gradle 파일의 플러그인 부분에 asciidoctor를 추가합니다.

build.gradle

```java
plugins {  
    // ...
    
    // asciidoctor  
    id 'org.asciidoctor.jvm.convert' version '3.3.2'  
}
```

동일하게 build.gradle 파일의 configurations 부분에 asciidoctorExt를 추가합니다. 이는 asciidoctor를 확장하는 종속성에 대한 구성을 선언한다는 의미입니다.

build.gradle
```java
configurations {  
    //...
    asciidoctorExt  
}
```

이번에는 dependencies 부분에 restdocs 관련 의존성을 추가합니다.
```java
dependencies { 
	// ... 
	// RestDocs 
	asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor' 
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc' 
}
```
- asciidoctorExit에 asciidoctor 의존성을 추가합니다.
- restdocs 생성시에 일반적으로 mockmvc를 사용하기 때문에 의존성을 추가합니다.

build.gradle 파일 마지막 부분에 다음 테스크 설정을 추가합니다.
```java
ext { // 전역변수
    snippetsDir = file('build/generated-snippets')
}

test {
    outputs.dir snippetsDir
}

asciidoctor {
    inputs.dir snippetsDir
    configurations 'asciidoctorExt'

    dependsOn test
}

bootJar {
    dependsOn asciidoctor
    from("${asciidoctor.outputDir}") {
        into 'static/docs'
    }
    //...
}
```
- ext는 전역변수를 선엄함을 의미합니다.
    - snippetsDir = file('build/generated-snippets') : snippetsDir 디렉토리에 대한 정의를 선언합니다. 문서 조각에 대한 경로를 위와 같이 정의합니다. 파일 타입의 build/generated-snippets 경로의 파일로 생성합니다.
- test 테스크에서는 테스트가 끝난 결과물을 snippetsDir 디렉토리에 저장함을 의미합니다.
- asciidoctor 테스크
    - “dependsOn test”의 의미는 test 테스크가 수행을 완료한 다음에 수행할 수 있다는 의미입니다.
    - inputs.dir snippetsDir의 의미는 snippetsDir 디렉토리를 입력으로 받아서 문서를 만든다는 의미입니다. configurations ‘aciidoctorExt’를 적용하여 플러그인 확장을 적용합니다.
- bootJar
    - asciidoctor 테스크가 수행된 다음에 수행됩니다.
    - from(”${asciidoctor.outputDir}”)을 통해서 나온 문서들을 static/docs 경로에 저장합니다.

### 2. Intellij AsciiDoc Plugin 추가
인텔리제이 IDE에서 asciidoc이라는 플러그인을 추가합니다. 해당 플러그인은 asciidoc 문법으로 작성한 결과를 볼 수 있는 플러그인입니다.

![[BE/메뉴얼/GCP/refImg/Pasted image 20240220125447.png]]

