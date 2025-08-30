
## 배경
- AWS RDS에서 GCP의 VM 인스턴스에서 동작하는 MySQL 컨테이너로 이전해야 합니다.
- RDS에서 fineAnts 스키마의 데이터를 덤프해야 합니다.

## MySQL 덤프하기
로컬 개발 환경에 mysql-client 설치
- 본인은 mac os이기 때문에 패키지 매니저로 home brew 사용
```shell
brew install mysql-client
```
