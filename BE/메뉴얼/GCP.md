
## 프로젝트 생성
- 프로젝트 이름 : fineants

프로젝트 생성 확인
![[Pasted image 20250827130244.png]]

## 서비스 계정 생성
서비스 계정은 사람이 아닌 애플리케이션이나 리소스가 GCP 리소스에 접근할 수 있도록 하는 인증 주체입니다. 보안, 권한 분리, 자동화된 인증을 위해서 서비스 계정이 사용됩니다.

IAM 서비스 이동 후 액세스 권한 부여 선택
![](BE/메뉴얼/refImg/Pasted%20image%2020250827135208.png)







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