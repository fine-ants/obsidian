
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
sudo certbot certonly --standalone -d services.fineants.co
```
- certonly : Certbot이 SSL 인증서를 발급받아 파일로 저장하지만, 이를 웹 서버(예: Apache, Nginx) 설정에 자동으로 적용하지 않습니다.
- --standalone : 웹 서버를 따로 설정할 필요 없이 Certbot이 **자체적으로 임시 웹 서버를 실행하여 도메인 소유권을 검증하는 방식
- -d : 도메인 이름을 설정합니다.
- 주의: services.fineants.co, services.release.fineants.co와 같이 도메인을 한꺼번에 설정하지 말고 각각 생성하세요.


SSL 인증서 발급에 성공하면 다음과 같은 결과가 나옵니다. /etc/letsencrypt/live/services.fineants.co-0002 디렉토리에 SSL 인증서가 저장됩니다.
![[Pasted image 20250106151029.png]]

**SSL 인증서 파일이 저장된 디렉토리로 이동**
```
su - root
password: {패스워드 입력}
root# cd /etc/letsencrypt/live/services.fineants.co-0002
```

**keystore.p12 파일 생성**
```
root#
sudo openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out keystore.p12 -name ttp -CAfile chain.pem -caname root
```
- 해당 명령어는 `openssl`을 사용하여 **PEM 형식의 SSL 인증서**를 **PKCS#12** 형식으로 변환하는 명령입니다.
- `openssl`은 공개키 암호화와 관련된 다양한 작업을 수행하는 툴입니다.
- openssl pkcs12 : **PKCS#12 형식**을 생성하거나 변환하는 명령어
	- PKCS#12는 인증서 및 개인 키, 인증서 체인 등을 하나의 파일로 패키징하는 표준 형식입니다. 일반적으로 `.p12` 또는 `.pfx` 확장자를 사용합니다.
- -export : 이 옵션은 **PKCS#12 파일로 내보내기**를 수행하는 옵션입니다. 즉, 인증서 및 관련 키를 하나의 `.p12` 파일로 묶겠다는 뜻입니다.
- `-in fullchain.pem` : `fullchain.pem` 파일은 **인증서**를 포함한 파일입니다. `fullchain.pem`은 서버 인증서와 중간 인증서를 포함하고 있어, 클라이언트가 인증서를 검증할 때 유용합니다. 이 파일이 포함된 인증서 체인은 SSL/TLS 핸드쉐이크 시 사용됩니다.
- -inkey privkey.pem : `privkey.pem`은 **개인 키**를 포함한 파일입니다. 이 개인 키는 SSL 인증서와 쌍을 이루며, 서버에서 인증서와 함께 사용됩니다. 이 옵션을 통해 개인 키를 PKCS#12 파일에 포함시킵니다.
- -out keystore.p12 : 이 옵션은 생성될 **PKCS#12 파일**의 출력 파일 이름을 지정합니다. 이 명령어에서는 `keystore.p12`라는 이름으로 출력됩니다. 이 파일은 SSL 인증서와 개인 키를 포함하고 있으며, Java 애플리케이션 등에서 사용될 수 있습니다.
- -name ttp : 이 옵션은 **PKCS#12 파일 내에서 인증서의 별칭(alias)**을 지정합니다. `ttp`는 이 인증서 항목의 이름으로, 나중에 해당 인증서를 참조할 때 사용됩니다.
- -CAfile chain.pem
	- `chain.pem` 파일은 **인증서 체인**을 포함하는 파일입니다. 이 파일에는 루트 인증서와 중간 인증서가 포함되어 있습니다. 클라이언트가 서버 인증서를 검증할 때 루트 인증서와 중간 인증서를 확인할 수 있도록 제공합니다.
	- `-CAfile` 옵션은 이 인증서 체인을 `PKCS#12` 파일에 포함시킵니다.
- -caname root : 이 옵션은 인증서 체인에서 루트 인증서를 참조하는 **이름을 지정**합니다. `root`는 루트 인증서를 지정하는 별칭입니다. 이 옵션은 체인에 포함된 루트 인증서의 이름을 정의합니다.

위 명령어 실행하면 비밀번호를 입력해야 합니다. 비밀번호를 입력하면 keystore.p12 파일이 생성됩니다. 
![[Pasted image 20250106151840.png]]

**keystore.p12 파일 생성확인**
```
ls
```
![[Pasted image 20250106151945.png]]

keysthore.p12 파일을 프로젝트의 디렉토리로 복사
다음 명령어는 ec2 인스턴스가 아닌 로컬 프로젝트가 존재하는 호스트 머신에서 수행합니다. 저같은 경우에는 mac os에서 수행됩니다.
```
scp -O -i "/Users/yonghwankim/.ssh/fineAnts.pem" root@3.35.207.14:/etc/letsencrypt/live/services.fineants.co-0002/keystore.p12 ~/Downloads
cp ~/Downloads/keystore.p12 /Users/yonghwankim/Documents/bootcamp/group/fintAnts/backend/src/main/resources/ssl/keystore.p12
```
- 첫번째 scp 명령어는 ec2 인스턴스의 keystore.p12 파일을 호스트 머신의 다운로드 디렉토리로 복사합니다.
- 두번째 cp 명령어는 호스트 머신의 다운로드 디렉토리에 있는 keystore.p12 파일을 프로젝트의 resources 디렉토리에 저장합니다.
- "fineAnts.pem" 파일은 ec2 인스턴스에 접속하기 위한 개인키 파일입니다. -i 옵션의 값으로 개인키 파일이 존재하는 경로를 적어야 합니다.

프로젝트에 keystore.p12 파일을 복사에 성공하면 커밋후 프로젝트를 다시 배포합니다.

---

## GCP 기반 SSL 인증서 발급

certbot 설치
```shell
sudo apt-get update
sudo apt-get install certbot
```

SSL 인증서 발급
```shell
sudo certbot certonly --standalone -d services.fineants.co
```

SSL 인증서가 발급된 디렉토리로 이동
```shell
su - root
password: {패스워드 입력}
cd /etc/letsencrypt/live/services.fineants.co
```
![](BE/문제해결/refImg/Pasted%20image%2020251111164125.png)

keystore.p12 파일 생성
```shell
sudo openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out keystore.p12 -name ttp -CAfile chain.pem -caname root
```
![](BE/문제해결/refImg/Pasted%20image%2020251111164221.png)

keysthore.p12 파일을 프로젝트의 디렉토리로 복사
```shell
scp -i ~/.ssh/gcp_vm \
    root@35.209.165.201:/etc/letsencrypt/live/services.fineants.co/keystore.p12 \
    ~/Downloads/

cp ~/Downloads/keystore.p12 /Users/yonghwankim/Documents/bootcamp/group/fintAnts/backend/src/main/resources/ssl/keystore.p12
```

#### root 계정 비밀번호 변경 후 접속하기
root 계정 접속 전 비밀번호 변경
```shell
sudo passwd root
```

SSH 설정 파일 수정
```shell
sudo vim /etc/ssh/sshd_config
```
![](BE/문제해결/refImg/Pasted%20image%2020251111164014.png)



---

## Spring Boot SSL 관련 설정
application-secret.yml 파일에서 SSL 설정
```
server:  
  ssl:  
    key-store: classpath:ssl/keystore.p12  
    key-store-type: PKCS12  
    key-store-password: {password}
```
- key-store-password 프로퍼티에 대한 비밀번호는 SSL 인증서 발급시 입력했던 비밀번호를 입력합니다.
- 해당 정보들은 시크릿 정보이기 때문에 외부에 노출하면 안됩니다.

docker-compose-production.yml 설정
```
version: "3.8"  
services:  
  app:  
    container_name: fineAnts_app  
    build: .  
    restart: always  
    ports:  
      - "443:443"
```
- docker-compose 설정에서 443 포트를 443 포트로 매핑합니다.
- spring 서버의 포트가 무조건 443포트로 설정하지는 않아도 되지만, 저는 고정하였습니다.
	- 그래서 별도의 application-production.yml 파일에서 443 포트로 설정하였습니다.

## 트러블 슈팅
### scp 복사 에러
**배경**
![[Pasted image 20250106154814.png]]

**원인**
- 이 에러는 sftp 클라이언트가 서버로부터 잘못된 응답을 수신했다는 의미입니다.
- sftp 서버와 클라이언트가 세션 수립 시, '.bashrc', '.profile' 등 서버의 쉘 기본 스크립트 (startup script)에서 발생하는 출력을 클라이언트가 sftp 메시지로 파싱하려 하기 때문에 에러가 발생합니다.

**해결방법**
sshd_config 파일을 열어서 다음과 같이 "Subsystem sftp /usr/libexec/openssh/sftp-server"를 주석 처리하고 다음과 같이 설정합니다.
```
ec2-user$ sudo vim /etc/ssh/sshd_config
```
![[Pasted image 20250106175513.png]]


sshd 서비스 재시작
```
ec2-user$ sudo service sshd restart
```

호스트 머신에서 다시 scp 명령어 수행
```
scp -i "/Users/yonghwankim/.ssh/fineAnts.pem" root@3.35.17.183:/etc/letsencrypt/live/services.fineants.co-0002/keystore.p12 ~/Downloads
```

복사한 실행 결과를 확인합니다.
![[Pasted image 20250106155548.png]]


### AWS CodeDeploy 배포 문제
배경
- AWS CodeDeploy에서 배포가 되지 않음
```
2025-01-06T09:38:23 ERROR [codedeploy-agent(2131)]: InstanceAgent::Plugins::CodeDeployPlugin::CommandPoller: Cannot reach InstanceService: Aws::CodeDeployCommand::Errors::AccessDeniedException - Aws::CodeDeployCommand::Errors::AccessDeniedException
```

원인
- 인스턴스에 기존 AWS 자격 증명 파일이 저장되어있어 IAM 정보를 제대로 못 가져오는 현상이 있을 수 있습니다.

해결방법
```bash
# AWS 자격증명 파일 삭제
$ sudo rm -rf /root/.aws/credentials

# codedeploy-agent 재시작
$ sudo systemctl restart codedeploy-agent
```

