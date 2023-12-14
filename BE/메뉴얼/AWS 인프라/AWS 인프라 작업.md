## 목차
- [[#VPC 생성]]
- [[#서브넷 생성]]
	- 서브넷 생성
	- 라우팅 테이블 연결 확인 
- [[#인터넷 게이트웨이 생성]]
	- 인터넷 게이트웨이 생성
	- VPC 연결
- [[#public subnet의 라우팅 테이블 생성]]
	- 라우팅 테이블 생성
	- 라우팅 테이블을 public subnet에 연결
- [[#public subnet의 ip 자동 할당 설정]]
- [[#EC2 인스턴스 생성]]
- [[#DB 서브넷 그룹 생성]]
- [[#데이터베이스 보안 그룹 생성]]
- [[#RDS 인스턴스 생성]]
- [[#EC2 인스턴스에서 RDS 데이터베이스 연결]]
- [[#IntelliJ에서 RDS 데이터베이스 원격 접속]]


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


## AWS CodeDeploy를 위한 S3 버킷 생성
AWS CodeDeploy를 이용하여 CI/CD 파이프 라인을 구축하기 위해서는 S3 버킷 생성이 필요합니다.

1. S3 서비스로 이동하여 버킷 생성을 합니다.
![[Pasted image 20231214172300.png]]

2. 버킷 생성 결과를 확인합니다.
![[Pasted image 20231214172323.png]]

3. 배포 zip 파일이 저장될 디렉토리를 생성합니다.
![[Pasted image 20231214172434.png]]

## AWS CodeDeploy를 위한 CodeDeploy 

1. 만약, CodeDeploy 서비스에 들어갔는데 권한이 필요하다면 해당 사용자 또는 그룹에 AWSCodeDeployFullAccess 권한을 얻습니다.
![[Pasted image 20231214172722.png]]

2. AWS CodeDeploy 서비스에서 배포에서 애플리케이션을 선택합니다. 그리고 애플리케이션 생성 버튼을 클릭합니다.
![[Pasted image 20231214172831.png]]

3. 애플리케이션에 대한 정보를 입력합니다.
![[Pasted image 20231214172850.png]]

4. 애플리케이션 생성 결과를 확인합니다.
![[Pasted image 20231214172936.png]]

5. fineAnts 애플리케이션을 들어가서 배포 그룹 탭으로 들어가서 배포 그룹 생성 버튼을 클릭합니다.
![[Pasted image 20231214173217.png]]

6. 배포 그룹 생성 정보를 입력합니다. 이 예제 같은 경우 개발 배포 서버에 배포하기 위해서 그룹을 생성하기 때문에 dev라고 명명하였습니다.
![[Pasted image 20231214173154.png]]

