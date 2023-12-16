## 목차
- [[#VPC 생성]]
- [[#서브넷 생성]] 
- [[#인터넷 게이트웨이 생성]]
- [[#public subnet의 라우팅 테이블 생성]]
- [[#public subnet의 ip 자동 할당 설정]]
- [[#EC2 인스턴스 생성]]
- [[#RDS 인스턴스 생성 전 사전 작업]]
- [[#DB 서브넷 그룹 생성]]
- [[#데이터베이스 보안 그룹 생성]]
- [[#RDS 인스턴스 생성]]
- [[#EC2 인스턴스에서 RDS 데이터베이스 연결]]
- [[#IntelliJ에서 RDS 데이터베이스 원격 접속]]
- [[#AWS CodeDeploy 사용하기]]
	- [[#IAM Role 생성]]
	- [[#EC2 인스턴스에 IAM 역할 적용]]
	- [[#Code Deploy Agent용 사용자 추가]]
	- [[#EC2에 Code Deploy Agent 설치]]
	- [[#프로젝트에 배포 파일 생성]]
	- [[#Code Deploy용 Role 생성]]
	- [[#Code Deploy 생성]]
	- [[#Code Deploy를 위한 S3 버킷 생성]]

## VPC 생성
1. VPC 대시보드에 입장하여 VPC 생성 버튼을 클릭합니다.
![[Pasted image 20231214143816.png]]

2. 다음과 같이 VPC 이름, IPv4 CIDR 블록을 입력합니다.
![[Pasted image 20231214143942.png]]
- 10.1.0.0/16을 설정함으로써 IPv4 주소의 앞의 16bit를 고정시킵니다.

3. VPC 생성 버튼을 클릭하여 fineAnts_vpc VPC를 생성합니다.
![[Pasted image 20231214144134.png]]

4. VPC 생성을 확인합니다.
![[Pasted image 20231214144210.png]]

## 서브넷 생성
1. 서브넷 메뉴에 입장하고 서브넷 생성 버튼을 클릭합니다.
![[Pasted image 20231214144526.png]]
![[Pasted image 20231214144547.png]]

2. 이전 단계에서 생성한 fineAnts_vpc를 선택하여 생성하려고 하는 서브넷이 fineAnts_vpc에 소속된다는 것을 설정합니다.
![[Pasted image 20231214145140.png]]

3. 우선 public subnet을 생성하기 위한 정보를 입력합니다.
![[Pasted image 20231214145345.png]]

4. 그 다음 새 서브넷 추가 버튼을 누르고 private subnet을 생성하기 위한 정보를 입력합니다.
![[Pasted image 20231214145541.png]]

5. 서브넷 생성 버튼을 클릭한다음 생성 결과를 확인합니다.
![[Pasted image 20231214145619.png]]

6. 서브넷의 라우팅 테이블을 확인하여 fineAnts_vpc의 CIDR인 10.1.0.0/16인지 확인합니다. fineAnts-public과 fineAnts-private 서브넷 ID를 클릭한 다음 라우팅 테이블 탭을 클릭합니다.
![[Pasted image 20231214145842.png]]
- 위와 같이 설정되어 있다면 fineAnts-public, fineAnts-private 서브넷은 fineAnts_vpc의 위치한 것임을 알 수 잇습니다.

## 인터넷 게이트웨이 생성
VPC에 인터넷 게이트웨이를 연결하지 않으면, VPC에서 인터넷과 연결되지 않습니다. 그래서 대부분의 경우 인터넷 게이트웨이를 VPC에 연결합니다.

1. 인터넷 게이트웨이 메뉴로 들어가서 인터넷 게이트웨이 생성 버튼을 클릭합니다.
![[Pasted image 20231214150124.png]]

2. 다음과 같이 인터넷 게이트웨이 정보를 입력하고 생성합니다.
![[Pasted image 20231214150351.png]]

3. 인터넷 게이트웨이 생성을 확인합니다.
![[Pasted image 20231214150417.png]]

4. fineAnts-igw 인터넷 게이트웨이를 fineAnts_vpc VPC에 연결하여 인터넷을 할 수 있게 합니다.
![[Pasted image 20231214150458.png]]
![[Pasted image 20231214150509.png]]
![[Pasted image 20231214150518.png]]

## public subnet의 라우팅 테이블 생성
public 서브넷이 인터넷과 연결하기 위한 라우팅 규칙이 필요합니다. public subnet 라우팅 테이블을 작성하고 public subnet과 연결합니다.

1. 라우팅 테이블 메뉴에 접속하고 라우팅 테이블 생성 버튼을 클릭합니다.
![[Pasted image 20231214150743.png]]

2. public subnet 라우팅 테이블에 대한 정보를 입력합니다.
![[Pasted image 20231214151043.png]]

3. fineAnts-public-rt 라우팅 테이블의 편집 버튼을 클릭하고 인터넷과 연결하기 위해서 다음과 같이 라우팅을 추가합니다.
![[Pasted image 20231214151354.png]]
![[Pasted image 20231214151421.png]]

4. 다음과 같이 fineAnts-public-rt 라우팅 테이블이 인터넷 게이트웨이와 연결하여 연결이 되었는지 확인합니다.
![[Pasted image 20231214151505.png]]

5. 서브넷 메뉴에 들어가고 fineAnts-public 서브넷의 서브넷 ID를 클릭합니다.
![[Pasted image 20231214151630.png]]

6. fineAnts-public 서브넷의 상세 페이지에서 라우팅 테이블 탭을 클릭하고 "라우팅 테이블 연결 편집" 버튼을 클릭합니다.
![[Pasted image 20231214151720.png]]

7. 다음과 같이 fineAnts-public 서브넷의 연결되고 있는 라우팅 테이블을 방금 생성한 fineAnts-public-rt 라우팅 테이블로 선택하고 저장 버튼을 클릭합니다.
![[Pasted image 20231214151810.png]]

8. fineAnts-public 서브넷의 라우팅 테이블이 fineAnts-public-rt 라우팅 테이블에 연결되었는지 확인합니다.
![[Pasted image 20231214151846.png]]

## public subnet의 ip 자동 할당 설정
1. 서브넷 메뉴에 입장하여 fineAnts-public 서브넷 ID를 클릭합니다.
![[Pasted image 20231214152223.png]]

2. 서브넷 상세 페이지에서 작업 -> 서브넷 설정 편집 버튼을 클릭하여 public subnet ip 자동 할당을 설정합니다.
![[Pasted image 20231214152315.png]]

3. 자동 할당 IP 설정에서 퍼블릭 IPv4 주소 자동 할당 활성화를 체크하고 저장합니다.
![[Pasted image 20231214152357.png]]


## EC2 인스턴스 생성

1. EC2 인스턴스 생성을 시작합니다. 우선 인스턴스에서 사용할 이름을 작성합니다.
![[Pasted image 20231214141703.png]]


EC2 인스턴스에 사용할 OS Image를 선택합니다. AWS 인프라와 호환성이 좋은 Amazon Linux를 선택합니다. AMI는 프리티어를 사용하기 때문에 프리티어에서 사용 가능한 Amazon Linux 2023 AMI를  선택합니다.
![[Pasted image 20231214141734.png]]

2. 다른 팀원들과 키 페어를 공유해야 하기 때문에 새로운 키 페어를 생성합니다. 다음 사진에서 "새 키 페어 생성" 버튼을 클릭합니다.
![[Pasted image 20231214142016.png]]

키 페어 이름을 입력하고 키 페어 생성 버튼을 클릭합니다.
![[Pasted image 20231214142001.png]]
- 키 페어 생성 버튼을 누르면 pem 키 파일이 다운로드 된것을 볼 수 있습니다.
- 키 파일을 분실되거나 노출되면 안되는 중요한 파일입니다.

3. 네트워크 설정에서 편집버튼을 클릭하여 이전 단계에서 생성한 fineAnts_vpc VPC와 외부 인터넷과 연결되기 위해서 fineAnts-public 서브넷을 선택합니다. 또한 보안 그룹을 별도로 생성하여 특정한 프로토콜과 포트만 연결되도록 티합니다.
![[Pasted image 20231214153809.png]]
- fineAnts-public-sg 보안 그룹 생성시 pem 키 파일을 이용한 원격 접속을 하기 위해서는 SSH를 추가해야 합니다.
![[Pasted image 20231214164941.png]]


4. 다른 설정은 기본값으로 두고 인스턴스 생성 버튼을 클릭합니다.
![[Pasted image 20231214142356.png]]

5. 인스턴스가 생성된 것을 확인합니다.
![[Pasted image 20231214142644.png]]


## RDS 인스턴스 생성 전 사전 작업
EC2 인스턴스와 RDS 인스턴스를 생성한 다음에 연결시키기 위해서는 우선 사전 작업이 필요합니다.
- VPC 생성
- 서브넷 생성
- 인터넷 게이트웨이 생성
- public subnet 라우팅 연결
- DB 서브넷 그룹 생성

위와 같은 사전 작업이 필요한 이유는 private subnet에 RDS를 위치시켜 외부에서 접근할 수 없도록 하고 오직 EC2 인스턴스를 통해서만 접근할 수 있도록 하기 위해서입니다.

## DB 서브넷 그룹 생성
DB 서브넷 그룹을 생성하기 위해서 최소 private한 서브넷이 2개 이상 필요합니다. 따라서 우선은 private 서브넷을 추가로 생성하겠습니다.

만약 DB 서브넷 그룹 생성시 한개의 private 서브넷만 가지고 생성하고자 한다면 다음과 같은 에러를 만나고 DB 서브넷 그룹이 생성되지 않습니다.
![[Pasted image 20231214160841.png]]

1. VPC의 서브넷 메뉴에 들어가서 서브넷 생성 버튼을 클릭합니다.
![[Pasted image 20231214160908.png]]

2. 새로운 private 서브넷을 생성하기 위한 정보를 입력합니다.
![[Pasted image 20231214160943.png]]
![[Pasted image 20231214161031.png]]
- 가용 영역을 fineAnts-private-1의 가용 영역(ap-northeast-2b)과 달라야 합니다. 어느 한쪽의 가용 영역이 이용 불가능해도 다른 가용 영역을 이용해서 데이터베이스의 데이터를 제공하기 위해서입니다.

3. 서브넷 생성 결과를 확인합니다.
![[Pasted image 20231214161237.png]]

4. fineAnts-private-2 private 서브넷에 private한 라우팅 테이블을 연결하기 위해서 라우팅 테이블을 생성합니다.
![[Pasted image 20231214161327.png]]
![[Pasted image 20231214161343.png]]
![[Pasted image 20231214161406.png]]

5. 다시 서브넷 메뉴로 돌아가서 fineAnts-private-2 서브넷 ID를 클릭하고 라우팅 테이블을 편집하여 fineAnts-private-rt 라우팅 테이블로 연결합니다.
![[Pasted image 20231214161452.png]]
![[Pasted image 20231214161513.png]]
![[Pasted image 20231214161523.png]]


6. RDS 서비스의 서브넷 그룹 메뉴에 접속합니다. 그리고 DB 서브넷 그룹 생성 버튼을 클릭합니다.
![[Pasted image 20231214161612.png]]

7. DB 서브넷 그룹을 생성하기 위한 정보를 입력합니다.
![[Pasted image 20231214161754.png]]
![[Pasted image 20231214161816.png]]
- 가용 영역 ap-northeast-2a, ap-northeast-2b 선택
- private subnet인 fineAnts-private-1, fineAnts-private-2 선택

8. DB 서브넷 그룹 생성을 확인합니다.
![[Pasted image 20231214161907.png]]


### 데이터베이스 보안 그룹 생성
1. EC2 서비스의 네트워크 및 보안에서 보안 그룹 메뉴를 선택하고 들어갑니다. 그리고 보안 그룹 생성 버튼을 클릭합니다.
![[Pasted image 20231214162150.png]]

2. 데이터베이스 보안 그룹의 정보를 입력합니다.
![[Pasted image 20231214162236.png]]

3. 인바운드 규칙을 정의합니다.
![[Pasted image 20231214162336.png]]

## RDS 인스턴스 생성
1. RDS 서비스로 이동하고 데이터베이스 메뉴로 입장하여 데이터베이스 생성 버튼을 클릭합니다.
![[Pasted image 20231214162520.png]]

2. 데이터베이스를 생성하기 위한 옵션을 선택합니다.
![[Pasted image 20231214162834.png]]
![[Pasted image 20231214162856.png]]
![[Pasted image 20231214162931.png]]
![[Pasted image 20231214162941.png]]
![[Pasted image 20231214163026.png]]
![[Pasted image 20231214163051.png]]

3. RDS 데이터베이스 생성 결과를 확인합니다.
![[Pasted image 20231214164018.png]]

## EC2 인스턴스에서 RDS 데이터베이스 연결
1. EC2 인스턴스에 접속합니다.
2. mysql을 설치합니다.
```
$ sudo yum update -y
$ wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
$ sudo yum localinstall mysql80-community-release-el9-1.noarch.rpm
$ sudo yum install mysql-community-server
$ sudo systemctl start mysqld
$ sudo systemctl status mysqld
```

![[Pasted image 20231214165227.png]]

3. RDS 서버에 원격 접속합니다.
```
$ mysql -u admin -h {RDS 주소} -p 
```
- RDS 주소는 RDS 서비스의 fineAnts 데이터베이스 상세 페이지에 들어가서 엔드 포인트를 확인하면 됩니다.
- 비밀번호는 RDS 데이터베이스 생성시 설정했던 비밀번호를 설정하면 됩니다.
![[Pasted image 20231214165404.png]]

4. mysql 원격 접속 결과를 확인합니다.
![[Pasted image 20231214165550.png]]

5. fineAnts 데이터베이스를 생성합니다.

```
mysql > create database fineAnts;
```

## IntelliJ에서 RDS 데이터베이스 원격 접속
1. 인텔리제이서 datasource 탭을 엽니다.
![[Pasted image 20231214165700.png]]
- 위 결과는 이미 접속한 것이 있기 때문에 접속된 것입니다.

2. 왼쪽 상단에 플러스 버튼을 클릭 -> Data Source -> MySQL을 선택합니다. 
![[Pasted image 20231214165742.png]]

3. 데이터소스의 이름을 작성하고 SSH/SSL 탭으로 이동합니다. 그리고 SSH Configuration에서 "..." 버튼을 클릭합니다.
![[Pasted image 20231214165945.png]]

4. 다음 그림과 같이 EC2 인스턴스의 IPv4 주소를 입력, 인증 타입(Authentication Type)을 키 페어 방식으로 선택한 다음 ec2 생성시 받았던 키 파일을 경로 선택합니다. 그리고 Test Connection 버튼을 클릭하여 EC2 인스턴스에 원격 접속이 되는지 테스트합니다.
![[Pasted image 20231214170142.png]]

5. 다음과 같이 RDS 정보를 입력합니다.
![[Pasted image 20231214170700.png]]
- HOST에는 RDS 데이터베이스의 엔드 포인트를 입력합니다.

6. Test Connection을 클릭하고 로그인 되면 OK를 누릅니다.
![[Pasted image 20231214170804.png]]


## AWS CodeDeploy 사용하기
### IAM Role 생성
CodeDeploy를 이용하여 배포하는 EC2 인스턴스가 S3와 CodeDeploy에 접근할 수 있는 권한을 부여하도록 역할을 생성해야 합니다.

1. IAM 서비스로 이동하여 역할 생성을 클릭합니다.
![[Pasted image 20231215142618.png]]

2. 다음과 같이 엔티티 유형과 사용사례를 선택합니다.
![[Pasted image 20231215142714.png]]
![[Pasted image 20231215142727.png]]

3. 다음을 클릭하고 역할에 부여하고  싶은 권한을 선택합니다. EC2가 S3와 CodeDeploy에 접근하기 위해서 다음과 같은 권한을 추가합니다.
```
- AmazonS3FullAccess
- AWSCodeDeployFullAccess
- AWSCodeDeployRole
- CloudWatchLogsFullAccess
```
- 마지막 CloudWatchLogsFullAccess는 추후 로깅 모니터링 때문에 추가하였습니다.

4. 역할 이름 및 추가한 권한을 다시 한번 확인합니다.
![[Pasted image 20231215143221.png]]
![[Pasted image 20231215143235.png]]

5. 역할 추가 결과를 확인합니다.
![[Pasted image 20231215143301.png]]

### EC2 인스턴스에 IAM 역할 적용
이전 단계에서 생성한 ec2-deploy를 EC2 인스턴스에 적용하여 배포할 수 있는 권한을 부여합니다.
1. EC2 서비스로 이동하여 생성된 EC2 인스턴스의 상세 페이지로 이도합니다. 그리고 작업 -> 보안 -> IAM 역할 수정을 클릭합니다.
![[Pasted image 20231215143848.png]]

2. ec2-deploy 역할을 선택하고 업데이트합니다.
![[Pasted image 20231215143918.png]]


### Code Deploy Agent용 사용자 추가
EC2 인스턴스가 Code Deploy 이벤트를 수신할 수 있도록 Agent를 설치해야 합니다. 그전에 EC2에서 AWS CLI를 사용할 수 있도록 IAM 사용자를 추가합니다.

1. IAM 서비스의 그룹 메뉴로 이동합니다. 그리고 그룹 생성 버튼을 클릭합니다.
![[Pasted image 20231215151331.png]]

2. 그룹이름을 deploy라고 입력하고 사용자 및 권한을 건들지 않고 생성합니다.
![[Pasted image 20231215151403.png]]

3. deploy 그룹 생성을 확인합니다.
![[Pasted image 20231215151428.png]]

4. deploy 그룹의 권한 탭 -> 권한 추가 -> 인라인 정책 생성 버튼을 클릭합니다.
![[Pasted image 20231215151505.png]]

5. 다음 사진과 같이 정책을 추가합니다.
![[Pasted image 20231215151746.png]]
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codedeploy:*",
                "ec2:*",
                "s3:*"
            ],
            "Resource": "*"
        }
    ]
}
```

6. 정책 이름을 입력하고 생성 버튼을 클릭합니다.
![[Pasted image 20231215151905.png]]

이제 deploy 그룹에 fineAnts 애플리케이션에 대한 배포 권한을 생성하였으니 deploy 그룹에 사용자를 추가해보겠습니다.

7. 사용자 메뉴로 이동하여 사용자 추가를 클릭합니다.
![[Pasted image 20231215152424.png]]

8. 배포용 사용자 이름을 입력합니다.
![[Pasted image 20231215152351.png]]

9. deploy 그룹에 fineAntsDeploy 사용자를 추가합니다.
![[Pasted image 20231215152522.png]]

10. 사용자 최종 권한을 확인합니다. 그리고 생성 버튼을 눌러서 사용자를 생성합니다.
![[Pasted image 20231215152600.png]]

11. 사용자 생성 결과를 확인합니다.
![[Pasted image 20231215152714.png]]

12. fineAntsDeploy 사용자의 상세 페이지로 들어가서 액세스 키를 생성합니다.
![[Pasted image 20231215152859.png]]

13. 사용 사례에서 Command Line Interface를 선택합니다.
![[Pasted image 20231215153034.png]]

14. 액세스 키 생성하고 csv 파일을 다운로드 받아둡니다.
![[Pasted image 20231215153456.png]]






### EC2에 Code Deploy Agent 설치
EC2 인스턴스에 CodeDeploy로 지정한 위치에서 파일을 받아 진행하기 위해서는 Code Deploy Agent가 설치되어야 합니다. 

1. EC2 접속 및 aws-cli를 설치합니다.
```
$ sudo yum install -y aws-cli
```

2. 사용자 홈 디렉토리로 이동하여 aws cli 설정을 합니다.
```
$ cd /home/ec2-user/
$ sudo aws configure
```

3. 다음과 같이 액세스 키 ID와 시크릿 키 값과 리전 정보, json을 입력합니다.
![[Pasted image 20231215154701.png]]

4. aws 설정이 끝났다면 Agent 설치 파일을 다운받습니다.
```bash
$ wget https://aws-codedeploy-ap-northeast-2.s3.amazonaws.com/latest/install
$ chmod +x ./install
$ sudo ./install auto
```

만약 install auto 명령어 실행시 루비(ruby)가 설치가 되어 있지 않으면 다음과 같은 에러가 발생합니다.
![[Pasted image 20231215155013.png]]

다음 명령어를 실행하여 루비를 설치합니다.
```
$ sudo yum install ruby
```

다시 Agent를 설치 시도합니다.
```
$ sudo ./install auto
```
![[Pasted image 20231215155511.png]]

5. 다음 명령어를 실행하여 EC2에 Agent가 실행중인지 확인합니다.
```bash
$ sudo service codedeploy-agent status
```
![[Pasted image 20231215155622.png]]

6. 마지막으로 EC2 인스턴스가 부팅되면 자동으로 AWS CodeDeploy Agent가 실행될 수 있도록 쉘 스크립트 파일을 생성합니다.
```bash
$ sudo vim /etc/init.d/codedeploy-startup.sh
```

쉘 스크립트 파일 내용은 다음과 같습니다.
```bash
#!/bin/bash 
echo 'Starting codedeploy-agent' 
sudo service codedeploy-agent restart
```

쉘 스크립트 파일 저장뒤 실행 권한을 추가합니다.
```bash
$ sudo chmod +x /etc/init.d/codedeploy-startup.sh
```

위 과정들을 통해서 EC2에 CodeDeploy Agent 설치가 완료되었습니다.

### 프로젝트에 배포 파일 생성
배포할 프로젝트의 환경은 Java11, Spring Boot, Gradle을 사용합니다. 이 단계에서는 프로젝트 디렉토리에 AWS CodeDeploy가 이용하여 배포할 스크립트를 작성합니다.

1. 프로젝트 최상단 위치에 appspec.yml 파일을 생성합니다.
![[Pasted image 20231215160958.png]]

2. appsepc.yml 파일에 다음과 같이 작성합니다.
```yml
version: 0.0  
os: linux  
files:  
  - source: /  
    destination: /home/ec2-user/build  
    overwrite: yes  
  
permissions:  
  - object: /  
    pattern: "**"  
    owner: ec2-user  
    group: ec2-user  
  
hooks:  
  ApplicationStart:  
    - location: scripts/start.sh  
      timeout: 60  
      runas: ec2-user
```
- AWS CodeDeploy는 appspec.yml 파일을 통해서 어떤 파일들을 어떤 위치로 배포하고, 이후 어떤 스크립트를 실행시킬 것인지 관리합니다.

```yml
os: linux  
files:  
  - source: /  
    destination: /home/ec2-user/build  
    overwrite: yes  
```
- 위 코드는 Code Build, S3, Github 등을 통해서 받은 전체 파일들(`source: /`)을 `/home/ec2-user/build/`로 옮기겠다는 의미입니다.

```yml
permissions:  
  - object: /  
    pattern: "**"  
    owner: ec2-user  
    group: ec2-user  
```
- `permissions`: 이 섹션은 파일 및 디렉토리에 대한 권한을 정의합니다.
- `object` : 권한을 부여할 대상을 정합니다. 여기서는 루트 디렉토리를 나타냅니다.
- `pattern` : 해당 대상에 대한 경로 패턴을 정의합니다. "**"는 임의의 하위 디렉토리를 나타냅니다. 따라서 모든 경로가 대상이 됩니다.
- `owner` : 대상 파일 또는 디렉터리의 소유자를 지정합니다. 여기서는 `ec2-user`로 설정되어 있습니다.
- `group`: 대상 파일 또는 디렉터리의 그룹을 지정합니다. 여기서도 `ec2-user`로 설정되어 있습니다.

```
hooks:  
  ApplicationStart:  
    - location: scripts/start.sh  
      timeout: 60  
      runas: ec2-user
```
- `hooks` : 이 섹션은 배포 중에 특정 이벤트가 발생했을 때 실행할 스크립트를 지정합니다.
- `ApplicationStart` : 이벤트의 이름을 나타냅니다. 여기서는 애플리케이션이 시작될 때 실행될 스크립트를 정의하고 있습니다.
- `location`: 실행할 스크립트의 위치를 지정합니다. 여기서는 scritps/start.sh 쉘 스크립트 파일을 실행합니다.
- `timeout` : 스크립트 실행 최대 시간. 스크립트가 시간을 초과하면 중단될 수 있습니다.
- `runas` : 스크립트를 실행할 사용자 지정. 스크립트는 해당 사용자 권한으로 실행됩니다.

2. scripts/start.sh 파일에 다음과 같이 작성합니다.
```
#!/bin/bash  
  
BUILD_JAR=$(ls /home/ec2-user/build/build/libs/*.jar)  
JAR_NAME=$(basename $BUILD_JAR)  
echo ">>> build filename: $JAR_NAME" >> /home/ec2-user/build/deploy.log  
  
echo ">>> copy build file" >> /home/ec2-user/build/deploy.log  
DEPLOY_PATH=/home/ec2-user/build/  
cp $BUILD_JAR $DEPLOY_PATH  
  
sudo chmod 666 /var/run/docker.sock  
sudo chmod +x /usr/local/bin/docker-compose  
docker-compose -f /home/ec2-user/build/docker-compose-dev.yml down -v  
docker-compose -f /home/ec2-user/build/docker-compose-dev.yml build  
docker-compose -f /home/ec2-user/build/docker-compose-dev.yml pull  
docker-compose -f /home/ec2-user/build/docker-compose-dev.yml up -d  
docker system prune -a
```


3. EC2에 `/home/ec2-user/build/` 디렉토리를 생성하고 프로젝트 세팅을 종료합니다.
```
$ mkdir /home/ec2-user/build/
```



### Code Deploy용 Role 생성
CodeDeploy를 통해서 EC2에 배포하기 위해서는 CodeDeploy가 EC2에 접근할 수 있도록 별도의 Role 생성을 해야 합니다.

1. IAM 서비스를 들어가서 역할 메뉴에 들어갑니다. 그리고 역할 생성 버튼을 클릭합니다.
![[Pasted image 20231215165808.png]]

2. 다음과 같이 AWS 서비스 및 CodeDeploy 사용 사례를 검색하여 선택합니다.
![[Pasted image 20231215170052.png]]

3. 추가된 권한을 확인합니다.
![[Pasted image 20231215170122.png]]

4. 역할 이름 및 설명을 추가합니다. 그리고 역할을 생성합니다.
![[Pasted image 20231215170301.png]]

5. 역할 생성 결과를 확인합니다.
![[Pasted image 20231215170349.png]]






### Code Deploy 생성
AWS Code Deploy 서비스에서 Code Deploy 애플리케이션을 생성하여 EC2에 배포하도록 합니다.

1. AWS CodeDeploy 서비스에서 배포에서 애플리케이션을 선택합니다. 그리고 애플리케이션 생성 버튼을 클릭합니다.
![[Pasted image 20231214172831.png]]

2. 애플리케이션에 대한 정보를 입력합니다.
![[Pasted image 20231214172850.png]]

4. 애플리케이션 생성 결과를 확인합니다.
![[Pasted image 20231214172936.png]]

5. fineAnts 애플리케이션을 들어가서 배포 그룹 탭으로 들어가서 배포 그룹 생성 버튼을 클릭합니다.
![[Pasted image 20231214173217.png]]

6. 배포 그룹 생성 정보를 입력합니다. 이 예제 같은 경우 개발 배포 서버에 배포하기 위해서 그룹을 생성하기 때문에 dev라고 명명하였습니다.
![[Pasted image 20231214173154.png]]

7. 이전 단계에서 생성한 Code Deploy용 Role을 선택합니다.
![[Pasted image 20231215171127.png]]

8. 배포 유형 및 환경 구성을 다음과 같이 설정합니다.
![[Pasted image 20231215171254.png]]
![[Pasted image 20231215171304.png]]
![[Pasted image 20231215171327.png]]
![[Pasted image 20231215171445.png]]

9. 배포 그룹 생성을 확인합니다.
![[Pasted image 20231215171837.png]]


### Code Deploy를 위한 S3 버킷 생성
배포 방식을 빌드하고 압축한 zip파일을 S3에 저장하였다가 EC2에 압축해제하면서 저장하기 위해서 S3 버킷을 생성합니다.

1. S3 서비스에 들어가서 버킷 생성 만들기합니다.
![[Pasted image 20231215173719.png]]

2. 버킷에 대한 정보를 입력합니다. 그리고 다른 설정들은 기본값으로 유지한채 생성합니다.
![[Pasted image 20231215173729.png]]

3. 버킷 생성 결과를 확인합니다.
![[Pasted image 20231215173819.png]]

4. fineants 버킷에 들어가서 deploy 폴더를 생성합니다. deploy 폴더에 배포한 zip 파일들을 저장할 것입니다.
![[Pasted image 20231215173836.png]]


위와 같은 과정을 통해서 AWS Code Deploy에 대한 환경 구성은 끝났습니다. 이제 Github Action 같은 툴을 통해서 AWS Code Deploy 서비스를 사용하여 EC2에 배포할 수 있습니다.




## Github Action을 이용한 CI/CD 구현
이전 글에서 EC2, RDS, Code Deploy를 생성하였습니다. 이제 CI/CD 과정을 통하여 EC2에 배포하기를 원합니다. 

CI/CD 구현과정을 소개하기에 앞서 이 글에서 소개하는 전체적인 인프라 구조는 다음과 같습니다.

![[fineAnts_architecture_v2.png]]
1. Github의 특정 브린치에 푸시됩니다.
2. Github Action에서 빌드하고 zip 파일로 압축하여 저장합니다.
3. Github Action에서 AWS Code Deploy에게 배포를 요청합니다.
4. AWS Code Deploy에서 S3에 저장된 zip 파일을 이용하여 EC2에 배포합니다.
5. EC2에서는 docker spring, redis 컨테이너를 실행합니다.

### Github Action Workflow 구현
1. Github Action을 수행하기 위해서 프로젝트 최상단을 기준으로 /.github/workflows 디렉토리에 cicd.yml 파일을 생성하고 코드를 작성합니다.
```yml
name: ci-cd  
  
on:  
  push:  
    branches: [ dev-cicd, dev ]  
  
permissions:  
  contents: read  
  
env:  
  S3_BUCKET_NAME: fineants  
  AWS_REGION: ap-northeast-2  
  CODEDEPLOY_NAME: fineAnts  
  CODEDEPLOY_GROUP: dev  
  
jobs:  
  build-image:  
    runs-on: ubuntu-latest  
    environment: dev  
    defaults:  
      run:  
        shell: bash  
  
    steps:  
      - uses: actions/checkout@v3  
        with:  
          submodules: true  
          token: ${{ secrets.GIT_TOKEN }}  
      ## JDK 설정  
      - name: Set up JDK 11  
        uses: actions/setup-java@v3  
        with:  
          java-version: '11'  
          distribution: 'temurin'  
      # gradle caching - 빌드 시간 향상  
      - name: Gradle Caching  
        uses: actions/cache@v3  
        with:  
          # 캐시할 디렉토리 경로를 지정합니다.  
          path: |  
            ~/.gradle/caches  
            ~/.gradle/wrapper  
          # 캐시를 구분하는 키를 지정합니다.  
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}  
          # 이전에 생성된 캐시를 복원하는데 사용할 키를 지정합니다.  
          # 캐시가 없거나 만료되었을때 이 키를 기반으로 이전에 생성된 캐시를 찾아 복원합니다.  
          restore-keys: |  
            ${{ runner.os }}-gradle-  
      # env 파일 생성  
      - name: make .env file  
        run: |  
          touch .env  
          echo "${{ secrets.ENV }}" > ./.env  
      # gradlew 실행을 위해서 실행 권한을 부여  
      - name: Grant execute permission for gradlew  
        run: chmod +x ./gradlew  
      # Gradle을 이용하여 빌드 수행  
      - name: Build with Gradle  
        run: ./gradlew bootJar  
      # zip 파일 생성  
      - name: Make zip file  
        run: zip -r ./$GITHUB_SHA.zip .  
      # AWS 인증정보 설정  
      - name: Configure AWS credentials  
        uses: aws-actions/configure-aws-credentials@v1  
        with:  
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}  
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}  
          aws-region: ${{ env.AWS_REGION }}  
      # S3에 업로드  
      - name: Upload to S3  
        run: aws s3 cp --region ap-northeast-2 ./$GITHUB_SHA.zip s3://$S3_BUCKET_NAME/$GITHUB_SHA.zip  
      # 코드 배포  
      - name: Code Deploy  
        run: aws deploy create-deployment --application-name $CODEDEPLOY_NAME --deployment-config-name CodeDeployDefault.AllAtOnce --deployment-group-name $CODEDEPLOY_GROUP --s3-location bucket=$S3_BUCKET_NAME,bundleType=zip,key=$GITHUB_SHA.zip
```

2. Github 시크릿 설정에서 환경변수를 설정합니다.
![[Pasted image 20231215232410.png]]


