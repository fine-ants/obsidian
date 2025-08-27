
## 프로젝트 생성
- 프로젝트 이름 : fineants

프로젝트 생성 확인
![[Pasted image 20250827130244.png]]

## 서비스 계정 생성
서비스 계정은 사람이 아닌 애플리케이션이나 리소스가 GCP 리소스에 접근할 수 있도록 하는 인증 주체입니다. 보안, 권한 분리, 자동화된 인증을 위해서 서비스 계정이 사용됩니다.

IAM 서비스 이동 후 서비스 계정 메뉴로 이동
![](BE/메뉴얼/refImg/Pasted%20image%2020250827135721.png)

"서비스 계정 만들기" 버튼을 클릭하여 새로운 서비스 계정을 생성
![](BE/메뉴얼/refImg/Pasted%20image%2020250827135819.png)


해당 서비스 계정은 Compute Engine에 접근하기 위해서 다음과 같이 역할을 선택
![](BE/메뉴얼/refImg/Pasted%20image%2020250827135934.png)

액세스 권한이 있는 주 구성원 단계에서는 별다른 선택하지 않고 완료합니다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827140006.png)

서비스 계정 생성 확인
![](BE/메뉴얼/refImg/Pasted%20image%2020250827140023.png)

## VPC 및 서브넷 환경 구성
VM 인스턴스가 연결할 VPC 및 서브넷을 생성합니다.

### VPC 생성
VPC 네트워크 서비스 이동
![](BE/메뉴얼/refImg/Pasted%20image%2020250827141923.png)

"VPC 네트워크 만들기" 버튼 클릭
![](BE/메뉴얼/refImg/Pasted%20image%2020250827142338.png)

VPC 네트워크 생성 정보 입력
![](BE/메뉴얼/refImg/Pasted%20image%2020250827142810.png)

public 서브넷 생성 정보 입력
![](BE/메뉴얼/refImg/Pasted%20image%2020250827143258.png)

![](BE/메뉴얼/refImg/Pasted%20image%2020250827143016.png)

private subnet 생성 정보 입력
![](BE/메뉴얼/refImg/Pasted%20image%2020250827143433.png)

![](BE/메뉴얼/refImg/Pasted%20image%2020250827143504.png)


고급 동적 라우팅 구성
![](BE/메뉴얼/refImg/Pasted%20image%2020250827143851.png)

VPC 및 서브넷 생성 확인
![](BE/메뉴얼/refImg/Pasted%20image%2020250827144120.png)
![](BE/메뉴얼/refImg/Pasted%20image%2020250827144136.png)

## 라우터 생성

Cloud Router 서비스로 이동합니다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827145328.png)

"Create router" 버튼을 클릭하여 라우터 생성을 진행합니다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827145810.png)

라우터 생성 정보 입력
![](BE/메뉴얼/refImg/Pasted%20image%2020250827151536.png)

라우터 생성 확인
![](BE/메뉴얼/refImg/Pasted%20image%2020250827151841.png)

## 프론트엔드 환경 구성
다음 링크에 접근하여 프로젝트를 생성할 예정입니다.
- link : https://console.firebase.google.com/

프로젝트 생성 정보 입력
![](BE/메뉴얼/refImg/Pasted%20image%2020250827153735.png)

Continue 선택
![](BE/메뉴얼/refImg/Pasted%20image%2020250827153800.png)
![](BE/메뉴얼/refImg/Pasted%20image%2020250827155308.png)

지역 설정
![](BE/메뉴얼/refImg/Pasted%20image%2020250827155459.png)

Firebase 프로젝트 생성 확인
![](BE/메뉴얼/refImg/Pasted%20image%2020250827155617.png)

왼쪽 사이드 메뉴의 Build -> Hosting 메뉴 접근
![](BE/메뉴얼/refImg/Pasted%20image%2020250827162155.png)

Get Started 클릭
![](BE/메뉴얼/refImg/Pasted%20image%2020250827162240.png)

Firebase CLI 설치
![](BE/메뉴얼/refImg/Pasted%20image%2020250827162915.png)

개발 환경에서 Firebase CLI을 설치하기 전에 Github Repository로부터 프론트엔드 프로젝트를 다운로드합니다.
```shell
git clone https://github.com/fine-ants/frontend
cd frontend
```

VS Code를 통해서 프론트엔드 프로젝트를 엽니다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827163605.png)

VS Code의 터미널을 열고 Firebase CLI을 설치합니다.
```shell
npm install -g firebase-tools
```
![](BE/메뉴얼/refImg/Pasted%20image%2020250827164237.png)

Firebase CLI 설치 확인
```
firebase --version
```
![](BE/메뉴얼/refImg/Pasted%20image%2020250827164525.png)

Firebase Google 로그인
- 로그인을 진행하다보면 구글 인증을 진행합니다.
```shell
firebase login
```
![](BE/메뉴얼/refImg/Pasted%20image%2020250827164839.png)
![](BE/메뉴얼/refImg/Pasted%20image%2020250827164902.png)

다음과 같이 콘솔에서 로그인이 되면 성공입니다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827164912.png)

프로젝트에서 Firebase 초기화
```shell
firebase init
```

Hosting 선택
- Hosting을 선택하기 위해서 스페이스바 키를 누름
![](BE/메뉴얼/refImg/Pasted%20image%2020250827165118.png)

기존 프로젝트 선택
![](BE/메뉴얼/refImg/Pasted%20image%2020250827165218.png)

build 입력후 엔터
![](BE/메뉴얼/refImg/Pasted%20image%2020250827165319.png)

싱글 페이지 앱으로 설정
- /index.html에 모든 URL들이 덮어쓰여진다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827165359.png)

Github 배포
![](BE/메뉴얼/refImg/Pasted%20image%2020250827165648.png)


Firebase 초기화 결과 확인
프로젝트의 파일들을 보면 firebase 파일이 2개 생성된 것을 확인할 수 있다.
![](BE/메뉴얼/refImg/Pasted%20image%2020250827165810.png)
- firebase.json : 호스팅 구성을 정의한다. 구성 정보에는 어떤 폴더를 기준으로 배포할지, 배포할 때 어떤 파일을 무시할지, 호스팅 대상 사이트는 어디에 있는지 등을 포함한다.
- .firebaserc : 배포 대상 프로젝트를 정의한다.




## todo
- 프론트엔드 배포 환경 구성
- 백엔드 환경 구성
	- Spring 서버
	- Redis
	- MySQL
- CI-CD 환경 구성
- 도메인 이전 설정

### 배포 환경 구성
- 프론트엔드
	- Firebase Hosting
- Spring Boot + Redis + MySQL
	- Compute Engine e2-micro 1대**(Free Tier 리전: `us-central1`)에 Docker Compose로 같이 실행


### GCP Free Tier 혜택
- Compute Engine e2-mirco 1개/월
- Cloud Storage 5GB/월