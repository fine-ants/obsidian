
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
sudo certbot certonly --standalone -d services.fineants.co -d services.release.fineants.co
```
- certonly : Certbot이 SSL 인증서를 발급받아 파일로 저장하지만, 이를 웹 서버(예: Apache, Nginx) 설정에 자동으로 적용하지 않습니다.
- --standalone : 웹 서버를 따로 설정할 필요 없이 Certbot이 **자체적으로 임시 웹 서버를 실행하여 도메인 소유권을 검증하는 방식
- -d : 도메인 이름을 설정합니다.

