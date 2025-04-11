
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
프로메테우스 설정 파일에서 rule.yml 파일을 참조하는데 다음 코드는 rule.yml 파일의 설정 내용중 일부입니다.
```yaml
# rule.yml  
groups:  
  - name: system-monitor  
    rules:  
      - alert: HighJvmMemoryUsage  
        expr: jvm_memory_used_bytes{area="heap"} / jvm_memory_max_bytes{area="heap"} > 0.9  
        for: 1m  
        labels:  
          severity: warning  
        annotations:  
          summary: "High JVM memory usage on {{ $labels.instance }}"  
          description: "{{ $labels.instance }} has a JVM memory usage above 90% (current value: {{ $value }})"
```

프로메테우스 설정
```yaml
# prometheus.yml  
global:  
  scrape_interval: 15s  
  scrape_timeout: 15s  
  evaluation_interval: 2m  
  external_labels:  
    monitor: 'system-monitor'  
  query_log_file: query_log_file.log  
alerting:  
  alertmanagers:  
    - static_configs:  
        - targets:  
            - 'fineAnts_alertmanager:9093'  
rule_files:  
  - "rule.yml"  
scrape_configs:  
  - job_name: "prometheus"  
    static_configs:  
      - targets:  
          - "prometheus:9090"  
  - job_name: "springboot"  
    metrics_path: "/actuator/prometheus"  
    scheme: "http"  
    scrape_interval: 5s  
    static_configs:  
      - targets:  
          - "host.docker.internal:8080"  
    #          - "app:8080"
    basic_auth:  
      username: "{username}"
      password: "{password}"
```

### alertmanager 설정
alertmanager 컨테이너가 참조할 설정파일입니다.
```yaml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: ‘{email}’
  smtp_auth_username: ‘{‘username}
  smtp_auth_password: ‘{password}’
route:
  receiver: 'email-receiver'
receivers:
  - name: 'email-receiver'
    email_configs:
      - to: ‘{수신 받을 email}’
```

docker-compose 설정
```yaml
version: "3.8"  
services:
	alertmanager:  
	  image: prom/alertmanager  
	  container_name: fineAnts_alertmanager  
	  volumes:  
	    - ./secret/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml  
	  command:  
	    - '--config.file=/etc/alertmanager/alertmanager.yml'  
	  ports:  
	    - "9093:9093"  
	  networks:  
	    - spring-net
```


oom 테스트 트리거
```java
@PostConstruct
public void triggerOOM() {
    new Thread(() -> {
        List<byte[]> memoryHog = new ArrayList<>();
        while (true) {
            memoryHog.add(new byte[1024 * 1024]); // 1MB씩 할당
            try {
                Thread.sleep(100);
            } catch (InterruptedException ignored) {}
        }
    }).start();
}

```

실행 결과는 다음과 같습니다.
![[Pasted image 20250411155730.png]]
