
## 도입 배경
- 프로메테우스를 배포하였지만 Spring 서버가 OOM(OutOfMemory) 발생시 별도의 알림이 없습니다.

## 프로메테우스 OOM 설정 적용하기
### 목표
Spring 서버에서 JVM 메모리 사용량이 너무 높거나, OutOfMemory가 발생할 때 프로메테우스가 이를 감지하고 AlertManager를 통해서 이메일로 알림 전송하는게 목적입니다.

### 사전 준비사항
- 프로메테우스 서버 운영 중
- Spring 서버가 프로메테우스와 통신할 수 있도록 Micrometer + Actuator + Prometheus exporter 설정되어 있어야 합니다.
- AlertManager와 연결되어 있어야 알림 전송이 가능합니다.

### Spring에서 JVM 메모리 메트릭 노출
`build.gradle` 의존성 추가
```gradle
implementation 'io.micrometer:micrometer-registry-prometheus'
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

`application.yaml`
```yaml
management:  
  endpoints:  
    web:  
      exposure:  
        include: # 노출할 Actuator 엔드포인트 목록  
          - prometheus  
          - health  
```


### 프로메테우스에서 JVM 메모리 사용량 감지
Spring Actuator는 다음과 같은 JVM 메모리 관련 메트릭을 제공합니다.
- `jvm_memory_used_bytes{area="heap"}`
- `jvm_memory_max_bytes{area="heap"}`

OOM 경고 Rule 예시
