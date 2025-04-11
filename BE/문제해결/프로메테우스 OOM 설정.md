
## 도입 배경
- 프로메테우스를 배포하였지만 Spring 서버가 OOM(OutOfMemory) 발생시 별도의 알림이 없습니다.

## 프로메테우스 OOM 설정 적용하기
### 목표
Spring 서버에서 JVM 메모리 사용량이 너무 높거나, OutOfMemory가 발생할 때 프로메테우스가 이를 감지하고 
