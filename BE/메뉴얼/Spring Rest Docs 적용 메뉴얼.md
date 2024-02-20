해당 문서는 Spring 프로젝트에서 Spring Rest Docs를 적용하기 위한 메뉴얼입니다.

### 1. asciidoctor 플러그인 추가
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

