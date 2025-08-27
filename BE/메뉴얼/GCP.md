
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