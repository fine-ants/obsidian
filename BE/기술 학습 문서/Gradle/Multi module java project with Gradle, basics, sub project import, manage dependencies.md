
## 개요
- 멀티 모듈 Gradle Project 생성 방법 학습
- 한개 이상의 모듈을 import 방법 학습

## 멀티 모듈 프로젝트는 무엇인가
멀티 모듈 프로젝트는 여러개의 작은 프로젝트들로 구성된 프로젝트입니다. 다음 프로젝트를 보면 프로젝트 이름은 "gradlebasics" 라는 이름이 프로젝트 이름이고 하나의 gradlebasics 라는 프로젝트 이름 아래에 sub-project-1, sub-project-2, common라는 모듈이 포함되어 있는 형태입니다. 물론 gradlebasics 라는 프로젝트 자체가 하나의 모듈로서 작동할 수도 있습니다.
![](refImg/Pasted%20image%2020251129152422.png)


## 루트 프로젝트 생성 및 모듈 생성
gradlebasics 프로젝트 생성
![](refImg/Pasted%20image%2020251129153119.png)

gradlebasics 프로젝트 생성 확인
```shell
./gradlew build
```
![](refImg/Pasted%20image%2020251129153217.png)

다음 화면을 보면 gradlebasics라는 프로젝트가 루트 프로젝트가 되는 것을 볼수 있습니다.
![](refImg/Pasted%20image%2020251129153226.png)

sub-project-1 모듈 생성
![](refImg/Pasted%20image%2020251129153352.png)

생성한 sub-project-1 디렉토리가 모듈로 취급하기 위해서 루트 프로젝트의 `settings.gradle` 파일을 수정하여 sub-project-1 모듈을 포함하도록 합니다.
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
