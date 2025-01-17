## 중앙 집중화 로깅의 필요성
### 기존 로그 수집 방식의 문제점
#### 1. 로그 확인의 불편함
기존까지는 도커 컨테이너 위에서 실행중인 서버가 생성하는 로그를 확인하기 위해서는 사용자가 직접 해당 컨테이너로 직접 접속해서 확인하고 있습니다. 다음 화면은 docker 명령어를 이용해서 fineAnts_app 컨테이너에 쉘 접속해서 로그를 확인하는 모습입니다.
![[Pasted image 20250109121655.png]]

#### 2. 분산 환경 문제
만약 현재 EC2 인스턴스만이 아닌 추가적인 EC2 인스턴스를 추가하여 스케일 아웃하게 되면 각각의 서버에서 생성하는 로그를 확인하기 어려워 집니다. 이는 첫번째 문제점으로도 이어집니다.

#### 3. 로그 저장 문제
각각의 서버에서 생성된 로그를 어딘가에 저장해야 하는데, 운용하는 서버가 많아지면 많아질수록 로그를 저장하는데 불편합니다.

위와 같은 문제점을 해결하기 위해서는 중앙 집중화 로깅 솔루션이 필요합니다. 해당 솔루션은 각각의 서버에서 발생하는 로그들을 수집한 다음에 어떤 한 저장소에 저장하고 대시보드와 같은 시각화 도구를 이용해서 모니터링하는 솔루션입니다.
![[Pasted image 20250109122155.png]]

## 다양한 중앙 집중화 로깅 솔루션
중앙 집중화 로깅 솔루션은 크게 3가지가 존재합니다.
1. AWS CloudWatch
	- AWS EC2와 연계가 쉽지만 요금이 발생합니다.
2. ELK(Elatic Search + Logstash + Kibana)
	- 강력한 검색 및 쿼리 기능을 제공
	- 무겁고 러닝 커브와 구축 및 운영 리소스가 있습니다.
3. PLG(Promtail + Loki + Grafana)
	- 대규모 데이터 처리는 ELK에 비해서 부족하지만 비교적 가볍고 설정이 어렵지 않고 요구되는 리소스가 적습니다.

이 글에서는 PLG 스택을 이용해서 중앙 집중화 로깅 솔루션을 구현합니다.

## PLG 스택
![[Pasted image 20250109122752.png]]
### Promtail: 로그 수집
- 서버 및 애플리케이션에서 발생한 로그를 수집하고 Grafana Loki에게 전송
- 로그를 구조화하고 필요한 메타 데이터를 추가하는 기능을 제공
	- Promtail이 원시 로그(raw log)를 처리하여 가독성과 분석 가능성을 높이는 형태로 변환하는 것을 의미합니다.
- Loki에서 쿼리를 효율적으로 수행할 수 있도록 라벨을 붙여주는 전처리 작업을 수행

### Loki: 로그 저장 및 인덱싱
- Loki는 로그를 저장하고 쿼리할 수 있는 로그 집계 시스템
- 라벨 기반의 메타데이터 인덱싱을 통해서 빠르고 비용에 효율적인 로그 검색을 지원합니다.
- Promtail로부터 수신한 로그를 라벨과 함께 저장하며, 필요시 `LogQL`을 통해 쿼리합니다.
- 다른 로그 집계 시스템(ELK, Graylog, Splunk, Fluented, Datadog Logs)과 비교해서 리소스를 절약합니다.
	- Loki는 색인(Index-Free Design)하지 않습니다. Loki는 로그 데이터를 색인하지 않고, 라벨로만 분류합니다. 이로 인해서 CPU와 메모리 사용량이 크게 줄어듭니다.
	- Loki는 저장소 사용량을 절약합니다. 로그 데이터를 압축된 상태로 절약하여 저장소 사용량을 최소화합니다. 로그 데이터를 원본 그대로 유지하므로 추가 색인 데이터가 없습니다.
	- Loki는 간단한 구성을 지원합니다. Prometheus의 메트릭 관리 방식과 비슷한 접근 방식을 사용하여 설정이 간단하고 리소스 사용이 효율적입니다.
- logfmt라는 로그 포맷팅을 이용해서 별도의 필드를 지정할 수 있습니다.

### Grafana: 로그 시각화 (대시보드)
- 시각화 도구
- Loki에서 가져온 로그를 시각화하고 다양한 쿼리를 사용해서 로그를 분석할 수 있습니다.
- 대시보드를 통해서 실시간 모니터링 및 알림 설정도 가능합니다.


## Grafana & Loki 환경 구성
docker를 이용해서 Grafana & Loki 서버를 구성할수도 있지만, Grafana Cloud를 이용하면 무료 플랜으로 환경을 구성할 수 있습니다.
- https://grafana.com/products/cloud/

1. 회원가입을 수행합니다.
2. Grafana Home 페이지로 이동한 다음에 다음과 같이 대시 보드를 생성을 진행합니다.
![[Pasted image 20250109135918.png]]
3. Add visualization 버튼을 클릭합니다.
![[Pasted image 20250109135955.png]]
4. 데이터 소스를 선택해야 합니다. Grafana Cloud Free Plan 선택시 기본적으로 제공하는 Loki 데이터 소스(grafanacloud-yonghwankimdev-logs)를 선택합니다.
![[Pasted image 20250109140250.png]]
5. "Save dashboard" 버튼을 클릭하여 대시보드를 저장합니다.
![[Pasted image 20250109140413.png]]

6. 대시보드 이름을 "fineAnts"로 작성하고 대시보드를 생성합니다. Home 메뉴로 이동하여 대시보드 생성을 확인합니다.
![[Pasted image 20250109140649.png]]
![[Pasted image 20250109141921.png]]

## Promtail 환경 구성
Promtail을 이용해서 로그를 수집해서 전처리하고 Loki로 전송하는 역할을 수행합니다.

### Promtail 설정 파일 구성
config.yml
```yml
server:  
  http_listen_port: 0  
  grpc_listen_port: 0  
  
positions:  
  filename: /tmp/positions.yaml  
  
clients:  
  - url: https://logs-prod-030.grafana.net/loki/api/v1/push  
    basic_auth:  
      username: "${LOKI_USERNAME}"  
      password: "${LOKI_PASSWORD}"  
scrape_configs:  
  - job_name: logs  # 통합된 job 이름  
    static_configs:  
      - targets:  
          - localhost  
        labels:  
          job: logs  
          log_level: info  # info 레벨 라벨 추가  
          __path__: /var/log/info/*.log  
      - targets:  
          - localhost  
        labels:  
          job: logs  
          log_level: warn  # warn 레벨 라벨 추가  
          __path__: /var/log/warn/*.log  
      - targets:  
          - localhost  
        labels:  
          job: logs  
          log_level: error  # error 레벨 라벨 추가  
          __path__: /var/log/error/*.log  
    pipeline_stages:  
      # traceId 추출 (모든 로그에서 추출 가능)  
      - regex:  
          expression: '\\[traceId=(?P<traceId>[^\\]]+)\\]'  
      # HTTP Request 관련 라벨 추출 (HTTP Request 메시지에서만 추출)  
      - regex:  
          expression: 'HTTPMethod=(?P<HTTPMethod>[A-Z]+) Path=(?P<Path>\S+) from IP=(?P<IP>[0-9a-fA-F:]+)'  
      # HTTP Response 관련 라벨 추출 (HTTP Response 메시지에서만 추출)  
      - regex:  
          expression: 'ResponseCode=(?P<ResponseCode>\\d+) ResponseMessage="(?P<ResponseMessage>[^"]+)"'  
      # ExecutionTime 추출 (ExecutionTime이 있는 메시지에서만 추출)  
      - regex:  
          expression: 'ExecutionTime=(?P<ExecutionTime>\\d+ms)'  
      - labels:  
          traceId: traceId  
          HTTPMethod: HTTPMethod  
          Path: Path  
          IP: IP  
          ResponseCode: ResponseCode  
          ResponseMessage: ResponseMessage  
          ResponseData: ResponseData  
          ExecutionTime: ExecutionTime
```



## 라벨링을 위한 로그 메시지 수정
## Grafana 대시보드 구성
