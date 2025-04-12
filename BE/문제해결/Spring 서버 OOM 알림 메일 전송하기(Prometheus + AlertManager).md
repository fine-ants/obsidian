
## 개요
fineAnts 프로젝트를 진행하던 중, Prometheus와 Grafana를 도입하여 JVM 상태를 시각화하고 모니터링 할 수 있었습니다. 그러나 메모리 부족(OutOfMemory, OOM)이나 인스턴스 다운과 같은 치명적인 상황 발생 시 별도의 알림 시스템이 없어 즉각적인 대응이 어려웠습니다.
이 글에서는 Spring 서버에서 OOM 상황이 발생할 때 관리자 이메일로 알림을 전송하는 방법을 정리합니다. Prometheus로 메모리 상태를 감지하고 AlertManager를 통해서 이메일 알림을 전송하는 구조입니다.

## 목표
Spring 애플리케이션의 JVM 힙 메모리 사용률이 90%를 초과하거나, OOM이 발생한 경우 관리자에게 이메일로 알림을 전송합니다.


## 사전 준비사항
- Prometheus 서버가 정상 작동 중이어야 합니다.
- Spring 애플리케이션이 Prometheus와 통신할 수 있도록 Micrometer 설정이 필요합니다.
- AlertManager가 Prometheus와 연결되어 있어야 합니다.
- 이메일 전송을 위한 SMTP 인증 정보가 준비되어 있어야 합니다.

## 1. Spring에서 JVM 메모리 메트릭 노출 설정
의존성 추가(`build.gradle`)
```gradle
implementation 'io.micrometer:micrometer-registry-prometheus'
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

`application.yaml` 설정
```yaml
management:  
  endpoints:  
    web:  
      exposure:  
        include: # 노출할 Actuator 엔드포인트 목록  
          - prometheus  
          - health  
```

## 2. Prometheus 알림 Rule 설정
`rule.yml` 예시
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

Prometheus 설정(prometheus.yml)
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
    scheme: "https"  
    scrape_interval: 5s  
    static_configs:  
      - targets:  
          - "services.fineants.co"
    basic_auth:  
      username: "{username}"
      password: "{password}"
```
- global.rule_files 설정을 이용해서 알림 설정이 저장된 rule.yml 파일을 참조합니다.
- alerting.alertmanagers.static_configs.targets

### 프로메테우스에서 JVM 메모리 사용량 감지
Spring Actuator는 다음과 같은 JVM 메모리 관련 메트릭을 제공합니다.
- `jvm_memory_used_bytes{area="heap"}`
- `jvm_memory_max_bytes{area="heap"}`

OOM 경고 Rule 설정하기
프로메테우스 설정 파일에서 rule.yml 파일을 참조하는데 다음 코드는 rule.yml 파일의 설정 내용중 일부입니다.


프로메테우스 설정


### alertmanager 설정
alertmanager 컨테이너가 참조할 설정파일입니다.
```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: ‘{email}’
  smtp_auth_username: ‘{username}‘
  smtp_auth_password: ‘{password}’
route:
  receiver: 'email-receiver'
receivers:
  - name: 'email-receiver'
    email_configs:
      - to: ‘{수신 받을 email}’
```
- email에는 이메일 전송하고자 하는 계정의 이메일을 작성합니다.
- username과 password에는 2단계 인증 후에 생성되는 username과 password를 작성합니다.
- 수신 받을 email에는 알림 발생시 이메일을 받을 관리자와 같은 이메일 주소를 작성합니다.

docker-compose 설정
docker-compose에 프로메테우스 및 alertmanager 서비스에 대해서 설정합니다.
```yaml
version: "3.8"  
services:
	prometheus:  
	  image: prom/prometheus:latest  
	  container_name: fineAnts_prometheus  
	  ports:  
	    - "9090:9090"  
	  command:  
	    - '--web.enable-lifecycle'  
	    - '--config.file=/etc/prometheus/prometheus.production.yml'  
	    - '--web.console.libraries=/etc/prometheus/console_libraries'  
	    - '--web.console.templates=/etc/prometheus/consoles'  
	  restart: always  
	  volumes:  
	    - ./secret/prometheus/config:/etc/prometheus  
	    - ./prometheus/volume:/prometheus  
	  networks:  
	    - spring-net
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


oom 테스트
다음 테스트는 로컬 환경에서 테스트하였습니다. Spring 코드에 다음과 같이 메서드를 정의한 다음에 서버를 실행시킵니다. 그러면 어느순간에 힙 사이즈가 초과했다고 에러가 발생할 것입니다.
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
