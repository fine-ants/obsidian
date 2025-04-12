
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
- alerting.alertmanagers.static_configs.targets 설정을 이용해서 알림을 수행하기 위한 alertmanager 서버의 호스트 및 포트를 설정합니다.

## 3. AlertManager 설정
`alertmanager.yml` 예시
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
- Gmail 사용시 앱 비밀번호를 발급받아 `smtp_auth_password`에 입력해야 합니다.

## 4. Docker Compose 구성
docker-compose 파일에서 프로메테우스 및 AlertManager 서비스에 대해서 구현합니다.
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


## 5. OOM 테스트 코드
테스트를 위해서 다음과 같은 메서드를 작성하면 의도적으로 JVM 메모리를 과다하게 사용해 OOM을 발생시킬 수 있습니다.
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

위 예제 코드를 실행시켜서 설정한 알림이 정상적으로 전송되면 다음과 같은 결과를 메일로 전송받습니다. 다음 결과는 알림 설정을 일부 수정하여 힙 사용률이 1%를 초과하면 알림이 가도록 임시적으로 수정해서 확인한 결과입니다.
![[Pasted image 20250411155730.png]]

## 결론
Prometheus + AlertManager 조합을 통해 Spring 서버의 OOM 상황을 실시간으로 감지하고 자동으로 알림을 전송할 수 있게 되었습니다. 이는 시스템 장애의 빠른 대응과 안정성 확보에 큰 도움이 됩니다. Spring 환경에서는 Micrometer와 Actuator를 통해 JVM 메트릭을 쉽게 수집할 수 있으며 Prometheus와 연동만으로 고도화된 모니터링과 알림을 구현할 수 있습니다.