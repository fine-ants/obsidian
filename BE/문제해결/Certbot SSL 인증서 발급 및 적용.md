
## Certbot 설치
ec2 amazone linux os에서 yum 패키지 매니저를 이용해서 certbot 프로그램을 설치합니다.
```
sudo yum update
sudo yum install certbot
```

certbot 프로그램을 설치하였으면 설치 확인합니다.
```
sudo certbot --version
```
![[Pasted image 20250106143846.png]]
- 위와 같이 버전이 출력되면 설치 성공입니다.

## SSL 인증서 발급
```
sudo certbot certonly
```
- certonly : Certbot이 SSL 인증서를 발급받아 파일로 저장하지만, 이를 웹 서버(예: Apache, Nginx) 설정에 자동으로 적용하지 않습니다.
![[Pasted image 20250106142446.png]]
위 화면에서 standalone인 2번을 선택합니다. 그리고 도메인 이름으로 services.fineants.co, services.release.fineants.co 두 도메인에 적용하기 위해서 다음과 같이 콤마로 구분하여 작성합니다.
![[Pasted image 20250106142808.png]]

