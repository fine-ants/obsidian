
- [[#매트릭 설정|매트릭 설정]]
	- [[#매트릭 설정#의존성 설정|의존성 설정]]
	- [[#매트릭 설정#actuator API 목록 확인|actuator API 목록 확인]]
	- [[#매트릭 설정#애플리케이션 프로퍼티 설정|애플리케이션 프로퍼티 설정]]
	- [[#매트릭 설정#프로메테우스 설정|프로메테우스 설정]]
	- [[#매트릭 설정#컨테이너 실행|컨테이너 실행]]
	- [[#매트릭 설정#그라파나 설정|그라파나 설정]]


## 매트릭 설정
### 의존성 설정
```gradle
implementation 'org.springframework.boot:spring-boot-starter-web' 
implementation 'org.springframework.boot:spring-boot-starter-actuator' 
implementation 'io.micrometer:micrometer-registry-prometheus'
```

### actuator API 목록 확인
위와 같이 의존성 설정 후 서버를 실행합니다. 그리고 "http://localhost:8080/actuator" 경로로 요청합니다. 실행 결과는 다음과 같습니다. 실행 결과를 보면 현재 actuator가 제공하는 엔드포인트 목록을 확인할 수 있습니다. 클라이언트는 필요한 메트릭을 제공하는 API에 접속해서 메트릭 데이터를 확인할 수 있습니다.
```json
{
	"_links": {
		"self": {
			"href": "http://localhost:8080/actuator",
			"templated": false
		},
		"health": {
			"href": "http://localhost:8080/actuator/health",
			"templated": false
		},
		"health-path": {
			"href": "http://localhost:8080/actuator/health/{*path}",
			"templated": true
		}
	}
}
```

### 애플리케이션 프로퍼티 설정
매트릭 관련 설정을 위해서 application.yaml 파일에 다음과 같이 설정합니다.
```yaml
management: 
  endpoints: 
    web: 
      exposure: 
        include: prometheus, health, info 
  metrics: 
    tags: 
      application: ${spring.application.name}
```
- management.endpoints.web.exposure.include
	- 외부에 노출할 엔드포인트를 지정합니다.
	- health, info 엔드포인트 외에도 prometheus 엔드포인트를 설정하여 해당 엔드포인트를 통해서 메트릭을 수집하도록 합니다.
- managment.metrics.tags.application
	- 메트릭 데이터에 태그를 추가하는 역할을 수행합니다.

위와 같이 설정 후 `/actuator/prometheus` 경로로 HTTP 요청하여 실행 결과를 확인합니다. 실행 결과를 보면 커넥션, 세션, 스레드 등과 같은 애플리케이션 메트릭 정보를 확인할 수 잇습니다.
![[Pasted image 20250228164726.png]]

### 프로메테우스 설정
```yaml
# prometheus.yml  
global:  
  scrape_interval: 15s  
  scrape_timeout: 15s  
  evaluation_interval: 2m  
  external_labels:  
    monitor: 'system-monitor'  
  query_log_file: query_log_file.log  
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
          - "services.fineants.co:443"
```
- global.scrape_interval : 기본적으로 모든 타겟을 15초마다 메트릭을 수집합니다.
- global.scrape_timeout : 15초 안에 응답하지 않은 경우 요청이 실패합니다.
- global.evaluation_interval : 2분마다 PromQL 쿼리를 실행하여 경고(rule.yml)에 대한 평가를 수행합니다.
- global.external_labels : 외부 레이블을 설정하여 Prometheus의 메트릭에 `monitor=system-monitor` 라벨을 추가합니다.
- global.query_log_file : PromQL 쿼리 로그를 `query_log_file.log` 파일에 기록합니다.
- rule_files : rule.yml 파일에서 경고 규칙(alerting rules) 및 레코드 규칙(recording rules)을 정의합니다.
- scrape_configs.static_configs.targets="prometheus:9090" : Prometheus 서버 자체를 "http://prometheus:9090"에서 모니터링합니다.

```yaml
  - job_name: "springboot"
    metrics_path: "/actuator/prometheus"
    scheme: "http"
    scrape_interval: 5s
    static_configs:
      - targets:
          - "fineAnts_app:8080"
```
- job_name="springboot"
	- Spring Boot 애플리케이션을 모니터링하는 작업입니다.
- metrics_path="/actuator/prometheus"
	- Spring Boot 애플리케이션에서 `http://fineAnts_app:8080/actuator/prometheus` 경로로 메트릭을 수집합니다.
- scheme="http"
	- http 프로토콜을 사용해서 메트릭을 가져옵니다.
- scrape_interval=5s
	- Spring Boot 애플레킹션의 매트릭을 5초마다 가져옵니다. (기본 설정인 15초보다 더 자주 수집)
- static_configs.targets
	- fineAnts_app:8080이란 컨테이너 또는 호스트에서 메트릭을 가져옵니다.

rule.yml
```yaml
# rule.yml  
groups:  
  - name: system-monitor  
    rules:  
      - alert: InstanceDown  
        expr: up == 0  
        for: 5m  
        labels:  
          serverity: page  
        annotations:  
          summary: "Instance {{ $labels.instance }} down"  
          description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."  
      - alert: APIHighRequestLatency  
        expr: api_http_request_latencies_second{quantile="0.5"} > 1  
        for: 10m  
        annotations:  
          summary: "High request latency on {{ $labels.instance }}"  
          description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```
- groups.name : 해당 그룹의 이름을 설정합니다.
- groups.rules : 그룹에 속하는 알람 규칙들을 정의합니다.
- groups.rules.alert : 알람의 이름을 설정합니다.
- groups.rules.expr : 알람이 발생하는 표현식을 정의합니다.
	- `up == 0`은 메트릭이 0인 경우를 표현하는 표현식입니다.
- groups.rules.for : 조건이 유지되어야 하는 시간을 설정합니다.
- groups.rules.labels : 알림에 대한 추가적인 레이블을 설정합니다.
- groups.rules.annotations : 알림에 대한 추가 설명을 제공하는 애노테이션을 설정합니다.

### 컨테이너 실행
```yaml
grafana:  
  container_name: fineAnts_grafana  
  image: grafana/grafana:11.4.0  
  ports:  
    - "3000:3000"  
  volumes:  
    - grafana_data:/var/lib/grafana  
  networks:  
    - spring-net  
prometheus:  
  image: prom/prometheus:latest  
  container_name: fineAnts_prometheus  
  ports:  
    - "9090:9090"  
  command:  
    - '--web.enable-lifecycle'  
    - '--config.file=/etc/prometheus/prometheus.yml'  
    - '--web.console.libraries=/etc/prometheus/console_libraries'  
    - '--web.console.templates=/etc/prometheus/consoles'  
  restart: always  
  volumes:  
    - ./prometheus/config/prod:/etc/prometheus  
    - ./prometheus/volume:/prometheus  
  networks:  
    - spring-net
```


### 그라파나 설정
그라파나의 관리자 계정으로 접속한 다음에 data source 메뉴로 이동합니다.
![[Pasted image 20250301153433.png]]


새로운 데이터 소스를 추가합니다. 검색창에 Prometheus를 검색하여 선택합니다.
![[Pasted image 20250301153454.png]]

URL에 프로메테우스의 호스트 주소와 포트주소를 입력합니다. 저 같은 경우 http://fineAnts_prometheus:9090 입니다.
![[Pasted image 20250301153509.png]]

Save & Test 버튼을 눌러서 연결을 확인한 다음에 생성 결과를 확인합니다.
![[Pasted image 20250301153543.png]]

이제는 Dashboards 메뉴로 이동한 다음에 Import 버튼을 클릭합니다.
![[Pasted image 20250301153610.png]]

다음과 같이 ID 입력 부분에 "4701"을 입력하고 Load합니다.
![[Pasted image 20250301153657.png]]

대시보드 생성 결과를 확인합니다.
![[Pasted image 20250301153710.png]]

