
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