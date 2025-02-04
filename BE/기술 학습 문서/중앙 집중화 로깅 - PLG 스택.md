- [[#중앙 집중화 로깅의 필요성|중앙 집중화 로깅의 필요성]]
	- [[#중앙 집중화 로깅의 필요성#기존 로그 수집 방식의 문제점|기존 로그 수집 방식의 문제점]]
		- [[#기존 로그 수집 방식의 문제점#1. 로그 확인의 불편함|1. 로그 확인의 불편함]]
		- [[#기존 로그 수집 방식의 문제점#2. 분산 환경 문제|2. 분산 환경 문제]]
		- [[#기존 로그 수집 방식의 문제점#3. 로그 저장 문제|3. 로그 저장 문제]]
- [[#다양한 중앙 집중화 로깅 솔루션|다양한 중앙 집중화 로깅 솔루션]]
- [[#PLG 스택|PLG 스택]]
	- [[#PLG 스택#Promtail: 로그 수집|Promtail: 로그 수집]]
	- [[#PLG 스택#Loki: 로그 저장 및 인덱싱|Loki: 로그 저장 및 인덱싱]]
	- [[#PLG 스택#Grafana: 로그 시각화 (대시보드)|Grafana: 로그 시각화 (대시보드)]]
- [[#Grafana & Loki 환경 구성|Grafana & Loki 환경 구성]]
- [[#Loki 환경 구성|Loki 환경 구성]]
- [[#Promtail 환경 구성|Promtail 환경 구성]]
	- [[#Promtail 환경 구성#Promtail 설정 파일 구성|Promtail 설정 파일 구성]]
- [[#라벨링을 위한 로그 메시지 수정|라벨링을 위한 로그 메시지 수정]]
- [[#Controller 로그 AOP 구현|Controller 로그 AOP 구현]]
- [[#Service 로그 AOP 구현|Service 로그 AOP 구현]]
- [[#Grafana 대시보드 구성|Grafana 대시보드 구성]]
	- [[#Grafana 대시보드 구성#HTTP Method 분포도|HTTP Method 분포도]]
	- [[#Grafana 대시보드 구성#상위 API 10개|상위 API 10개]]
	- [[#Grafana 대시보드 구성#IP 분포도 상위 10개|IP 분포도 상위 10개]]
	- [[#Grafana 대시보드 구성#예외 발생 로그 보기|예외 발생 로그 보기]]
	- [[#Grafana 대시보드 구성#API별 예외 발생 빈도|API별 예외 발생 빈도]]
	- [[#Grafana 대시보드 구성#API별 평균 실행 시간|API별 평균 실행 시간]]
- [[#docker-compose 설정|docker-compose 설정]]
- [[#Logback Log Rotation 설정|Logback Log Rotation 설정]]
- [[#Amazon S3 저장소로 내보내기(Export to Amazon S3)|Amazon S3 저장소로 내보내기(Export to Amazon S3)]]
	- [[#Amazon S3 저장소로 내보내기(Export to Amazon S3)#사전 작업|사전 작업]]
	- [[#Amazon S3 저장소로 내보내기(Export to Amazon S3)#수행 과정|수행 과정]]


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

## Loki 환경 구성
Loki 도커 컨테이너를 이용해서 로그를 저장하는 역할을 수행합니다.
프로덕션 환경에서 실행할 docker-compose.yml 파일을 다음과 같이 구현합니다.
```
version: "3.8"  
services:  
  # ...
  
  loki:  
  container_name: fineAnts_loki  
  image: grafana/loki  
  ports:  
    - "3100:3100"  
  volumes:  
    - ./loki:/etc/loki  
    - loki_data:/loki  
  command: -config.file=/etc/loki/config.yaml  
  networks:  
    - spring-net
```

loki/config.yaml
```yml
# 인증 기능 비활성화  
auth_enabled: false  
  
server:  
  # loki 서버가 수신할 포트  
  http_listen_port: 3100  
  
ingester:  
  lifecycler:
	# Loki ingester 인스턴스를 구분할 주소
    address: fineAnts_loki  
    ring:  
      kvstore:  
	    # 상태 정보를 메모리에서만 저장
        store: inmemory  
	    # 데이터 복제 팩터 설정, 데이터가 1개의 인스턴스에만 저장됨
      replication_factor: 1  
  # 로그 청크가 비활성 상태로 대기하는 시간, 이 시간동안 로그가 추가되지 않으면 해당 청크는 종료됩니다.
  # 5분동안 로그가 추가되지 않으면 청크가 종료되어 저장됨
  chunk_idle_period: 5m  
  # 청크 저장후 데이터를 얼마나 오래 유지할지 
  chunk_retain_period: 30s  
  wal:  
    dir: /loki/wal  
  
schema_config:  
  configs:  
    - from: 2020-10-24  
      store: boltdb  
      object_store: filesystem  
      schema: v11  
      index:  
        prefix: index_  
        period: 168h  
  
storage_config:  
  boltdb:  
    directory: /loki/index  
  filesystem:  
    directory: /loki/chunks  
  
limits_config:  
  allow_structured_metadata: false  
  reject_old_samples: true  
  reject_old_samples_max_age: 168h
```
- Loki의 `ingester` 는 로그를 수집하여 저장하는 역할을 수행합니다.
- KV store는 ingester의 **상태 관리와 클러스터링**에 중요한 역할을 하는 데이터 저장소입니다. ingester의 ring(클러스터링 상태)을 관리하거나 데이터 복제와 정합성을 유지하는데 사용됩니다. 분산 환경에서 여러 ingester 인스턴스들이 서로 협업할 수 있도록 합니다.


References
- https://velog.io/@roycewon/Promtail-Loki%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%9C-Logback-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81

## Grafana 컨테이너 환경 구성
```
version: "3.8"  
services:
	grafana:  
	  container_name: fineAnts_grafana  
	  image: grafana/grafana:latest  
	  ports:  
	    - "3000:3000"  
	  volumes:  
		- grafana_data:/var/lib/grafana
	  networks:  
	    - spring-net
volumes:  
  grafana_data:
```

로키 및 그라파나 컨테이너를 실행한 다음에 로컬 환경에서 실행시 `http://localhost:3000` 으로 접속합니다. 그리고 email:admin password: admin으로 접속합니다.
![[Pasted image 20250203142532.png]]

사이드 메뉴의 Connections -> Add New Connection 메뉴를 클릭합니다.
![[Pasted image 20250203142620.png]]

"loki"를 검색한 다음에 데이터 소스를 추가합니다. 다음 화면에서 Add new data source 버튼을 클릭합니다.
![[Pasted image 20250203142643.png]]

다음과 같이 데이터 소스 이름과 URL을 입력합니다.
![[Pasted image 20250203142745.png]]
- 현재 로컬 환경에서 loki 컨테이너와 grafana 컨테이너가 같은 네트워크에서 동작하기 때문에 http://localhost:3100 으로 요청하면 연결되지 않고 컨테이너 이름을 통해서 연결해야 합니다. localhost로 지정하면 grfana 컨테이너 내부의 localhost를 지정하기 때문입니다.

다음과 같이 Derived fields 메뉴에서 traceID 필드를 추가합니다.
![[Pasted image 20250203143653.png]]


하단에 저장 및 테스트 버튼을 클릭하여 결과를 확인합니다. 다음과 같이 연결이 성공하면 정상입니다.
![[Pasted image 20250203142933.png]]


대시보드 메뉴로 들어가서 다음 json 대시보드를 import합니다.
![[Pasted image 20250203145858.png]]

다음 화면에서 json 파일을 업로드합니다.
![[fineAnts-1738561890350.json]]

![[Pasted image 20250203145914.png]]

대시보드 import 결과를 확인합니다. 각각의 패널의 데이터 소스를 fineAnts_loki 데이터 소스로 설정하고 한번 쿼리를 해주어야 합니다.
![[Pasted image 20250203150313.png]]


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
promtail 프로세스에서 라벨링을 하기 위해서 설정 파일에 맞게 로깅 설정합니다. 이 문서 같은 경우에는 spring 프레임워크 기반에 로그백(logback)을 사용합니다.

loback-spring.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!-- 60초마다 설정 파일의 변경을 확인 하여 변경시 갱신 -->  
<configuration scan="true" scanPeriod="60 seconds">  
    <include resource="logback/appender/console/console_appender.xml"/>  
    <include resource="logback/appender/file/info_file_appender.xml"/>  
    <include resource="logback/appender/file/debug_file_appender.xml"/>  
    <include resource="logback/appender/file/warn_file_appender.xml"/>  
    <include resource="logback/appender/file/error_file_appender.xml"/>  
    <include resource="logback/property/property.xml"/>  
  
    <springProfile name="local">  
        <logger name="co.fineants" level="DEBUG" additivity="false">  
            <appender-ref ref="CONSOLE"/>  
            <appender-ref ref="INFO_FILE"/>  
            <appender-ref ref="DEBUG_FILE"/>  
            <appender-ref ref="WARN_FILE"/>  
            <appender-ref ref="ERROR_FILE"/>  
        </logger>    </springProfile>  
    <springProfile name="release, production">  
        <logger name="co.fineants" level="INFO" additivity="false">  
            <appender-ref ref="CONSOLE"/>  
            <appender-ref ref="INFO_FILE"/>  
            <appender-ref ref="WARN_FILE"/>  
            <appender-ref ref="ERROR_FILE"/>  
        </logger>    
	</springProfile>
</configuration>
```

property.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<included>  
    <!-- log file path -->  
    <property name="LOG_PATH" value="logs/"/>  
    <!-- info log file name -->  
    <property name="INFO_LOG_FILE_NAME" value="info"/>  
    <!-- debug log file name -->  
    <property name="DEBUG_LOG_FILE_NAME" value="debug"/>  
    <!-- warn log file name -->  
    <property name="WARN_LOG_FILE_NAME" value="warn"/>  
    <!-- err log file name -->  
    <property name="ERROR_LOG_FILE_NAME" value="error"/>  
    <!-- pattern -->  
    <property name="LOG_PATTERN"  
              value="%d{ISO8601} %-5level [%thread] [traceId=%X{traceId}] %logger{36}:%line - %msg %n"/>  
  
    <!-- 개별 로그 레벨에 맞는 경로 -->  
    <property name="INFO_LOG_FILE_PATTERN" value="${LOG_PATH}/info/info.%d{yyyy-MM-dd}_%i.log"/>  
    <property name="DEBUG_LOG_FILE_PATTERN" value="${LOG_PATH}/debug/debug.%d{yyyy-MM-dd}_%i.log"/>  
    <property name="WARN_LOG_FILE_PATTERN" value="${LOG_PATH}/warn/warn.%d{yyyy-MM-dd}_%i.log"/>  
    <property name="ERROR_LOG_FILE_PATTERN" value="${LOG_PATH}/error/error.%d{yyyy-MM-dd}_%i.log"/>  
    <!-- 최대 파일 사이즈 -->  
    <property name="MAX_FILE_SIZE" value="1GB"/>  
    <!-- 최대 내역 일수 -->  
    <property name="MAX_HISTORY_DAYS" value="7"/>  
    <!-- CHARSET -->  
    <property name="CHARSET" value="UTF-8"/>  
</included>
```

info_file_appender.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<included>  
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>info</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>        <file>${LOG_PATH}/info/${INFO_LOG_FILE_NAME}.log</file>  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <!-- 이 옵션이 없을 경우 한글이 깨지는 경우 있음-->  
            <charset>${CHARSET}</charset>  
            <pattern>${LOG_PATTERN}</pattern>  
        </encoder>        <!-- Rolling 정책 -->  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->  
            <fileNamePattern>${INFO_LOG_FILE_PATTERN}</fileNamePattern>  
            <timeBasedFileNamingAndTriggeringPolicy                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
                <!-- 파일당 최고 용량 kb, mb, gb -->                <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>  
            </timeBasedFileNamingAndTriggeringPolicy>            <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->  
            <maxHistory>${MAX_HISTORY_DAYS}</maxHistory>  
        </rollingPolicy>    </appender></included>
```

나머지 warn_file_appender.xml, debug_file_appender.xml, error_file_appender.xml 파일에 대해서도 위 설정과 비슷하여 생략합니다.

console_appender.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<included>  
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <charset>${CHARSET}</charset>  
            <pattern>${LOG_PATTERN}</pattern>  
        </encoder>    </appender></included>
```

## Controller 로그 AOP 구현
Controller 레이어로 들어오는 HttpRequest에 대해서 다음 코드를 구현하여 로깅하도록 합니다.
```java
@Component  
@Aspect  
@Slf4j  
@Profile("!test")  
public class HttpLoggingAspect {  
    private static final String HTTP_REQUEST_LOG_FORMAT = "HTTP Request: HTTPMethod={} Path={} from IP={}";  
    private static final String HTTP_RESPONSE_LOG_FORMAT =  
       "HTTP Response: Path={} ResponseCode={} ResponseMessage=\"{}\" ResponseData=\"{}\"";  
    private static final String HTTP_EXECUTION_LOG_FORMAT = "HTTP Execution: Path={} ExecutionTime={}ms";  
    private long startTime;  
  
    // Controller의 모든 메서드에 대해 적용  
  
    @Pointcut("execution(* co.fineants..controller.*.*(..))")  
    public void pointCut() {  
  
    }  
  
    @Before("pointCut()")  
    public void logHttpRequest(JoinPoint ignoredJoinPoint) {  
       startTime = System.currentTimeMillis();  
       HttpServletRequest request = ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes())  
          .getRequest();  
       log.info(HTTP_REQUEST_LOG_FORMAT, request.getMethod(), request.getRequestURL(), request.getRemoteAddr());  
    }  
  
    // 메서드 호출 후 정상적으로 반환된 경우 로그 남기기  
    @AfterReturning(pointcut = "pointCut()", returning = "response")  
    public void logAfterReturning(JoinPoint ignoredJoinPoint, ApiResponse<?> response) {  
       HttpServletRequest request =  
          ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();  
       log.info(HTTP_RESPONSE_LOG_FORMAT, request.getRequestURI(),  
          response.getCode(), response.getMessage(), response.getData());  
    }  
  
    // 예외 발생시 로그 남기기  
    @AfterThrowing(pointcut = "pointCut()", throwing = "ex")  
    public void logAfterThrowing(JoinPoint ignoredJoinPoint, Throwable ex) {  
       HttpServletRequest request =  
          ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();  
       // 특정 비즈니스 예외 처리  
       if (ex instanceof FineAntsException fineAntsException) {  
          log.warn(HTTP_RESPONSE_LOG_FORMAT,  
             request.getRequestURI(), fineAntsException.getHttpStatusCode(), fineAntsException.getMessage(), null,  
             ex);  
       } else if (ex instanceof MethodArgumentNotValidException methodArgumentNotValidException) {  
          String errorMessage = Objects.requireNonNull(methodArgumentNotValidException.getBindingResult()  
             .getFieldError()).getDefaultMessage();  
          log.warn(HTTP_RESPONSE_LOG_FORMAT,  
             request.getRequestURI(), HttpStatus.BAD_REQUEST.value(), errorMessage, null, ex);  
       } else if (ex instanceof MissingServletRequestPartException missingServletRequestPartException) {  
          String errorMessage = missingServletRequestPartException.getMessage();  
          log.warn(HTTP_RESPONSE_LOG_FORMAT,  
             request.getRequestURI(), HttpStatus.BAD_REQUEST.value(), errorMessage, null, ex);  
       } else {  
          log.error(HTTP_RESPONSE_LOG_FORMAT,  
             request.getRequestURI(), HttpStatus.INTERNAL_SERVER_ERROR.value(), ex.getMessage(), null, ex);  
       }  
    }  
  
    // 완전히 종료된후 메서드 실행시간 측정하기  
    @After("pointCut()")  
    public void logAfter(JoinPoint ignoredJoinPoint) {  
       HttpServletRequest request =  
          ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();  
       long executionTime = System.currentTimeMillis() - startTime;  
       log.info(HTTP_EXECUTION_LOG_FORMAT, request.getRequestURI(), executionTime);  
    }  
}
```

## Service 로그 AOP 구현
서비스 레이어의 메소드 수행시 실행 정보들을 로깅하기 위해서 다음과 같이 구현합니다.
```java
@Component  
@Aspect  
@Slf4j  
@Profile("!test")  
public class ServiceLogAspect {  
    private long startTime;  
  
    // service의 모든 메서드에 대해 적용  
    @Pointcut("execution(* co.fineants..service.*.*(..))")  
    public void pointCut() {  
  
    }  
  
    // 메서드 호출 전 로그 남기기  
    @Before("pointCut()")  
    public void logBefore(JoinPoint joinPoint) {  
       startTime = System.currentTimeMillis();  
       String methodName = ((MethodSignature)joinPoint.getSignature()).getMethod().getName();  
       String args = Arrays.toString(joinPoint.getArgs());  
       log.info("Entering Service: Method={} with Args={}", methodName, args);  
    }  
  
    // 메서드 호출 후 정상적으로 반환된 경우 로그 남기기  
    @AfterReturning(pointcut = "pointCut()", returning = "result")  
    public void logAfterReturning(JoinPoint joinPoint, Object result) {  
       String methodName = ((MethodSignature)joinPoint.getSignature()).getMethod().getName();  
       log.info("Exiting Service: Method={}, with Return={}", methodName, result);  
    }  
  
    // 완전히 종료된후 메서드 실행시간 측정하기  
    @After("pointCut()")  
    public void logAfter(JoinPoint joinPoint) {  
       long executionTime = System.currentTimeMillis() - startTime;  
       String methodName = ((MethodSignature)joinPoint.getSignature()).getMethod().getName();  
       log.info("Method={}, ExecutionTime={}ms", methodName, executionTime);  
    }  
}
```

## Grafana 대시보드 구성
### HTTP Method 분포도
![[Pasted image 20250117142636.png]]
```
sum by(HTTPMethod) (count_over_time({job="logs"} | HTTPMethod != `` [$__auto]))
```
- Visualization : Pie Chart

### 상위 API 10개
![[Pasted image 20250117142839.png]]
```
topk(10, sum by(Path) (count_over_time({job="logs"} | Path != `` | HTTPMethod != `` [$__auto])))
```

### IP 분포도 상위 10개
![[Pasted image 20250117142947.png]]```
```
topk(10, sum by(IP) (count_over_time({job="logs"} | IP != `` [$__auto])))
```

### 예외 발생 로그 보기
![[Pasted image 20250117143156.png]]
```
{job="logs", log_level="error"} | traceId != "" | line_format `{{.traceId}}`
```
- Visualization : Table
- Time Data Link : `/d/${dashboard_uid}/${dashboard_slug}?orgId=1&var-traceId=${__data.fields["TraceId"]}`
- TraceId Data Link : `/d/${dashboard_uid}/${dashboard_slug}?orgId=1&var-traceId=${__value.text}`

![[Pasted image 20250117143503.png]]
- Visualization : Table
- Time Data Link : `/d/${dashboard_uid}/${dashboard_slug}?orgId=1&var-traceId=${__data.fields["TraceId"]}`
- TraceId Data Link : `/d/${dashboard_uid}/${dashboard_slug}?orgId=1&var-traceId=${__value.text}`

![[Pasted image 20250117143557.png]]
- 특정한 TraceId이나 Time 레코드를 선택시 클릭한 TraceId 일치하는 로그들을 위와 같이 출력해줍니다.
- Visualization : Logs
```
{traceId="$traceId"}
```

### API별 예외 발생 빈도
![[Pasted image 20250117152713.png]]
```
topk(10, sum by(Path) (count_over_time({job="logs", log_level=~"warn|error"} | logfmt | Path != `` [$__auto])))
```
- Visualization : Bar Chart
- X Axis : Path
- Orientation : Horizontal

### API별 평균 실행 시간
![[Pasted image 20250117174246.png]]
```
avg_over_time({job="logs"}
| regexp "Path=(?P<Path>\\S+) ExecutionTime=(?P<TempExecutionTime>\\d+)ms"
| Path != ``
| TempExecutionTime != ``
| unwrap TempExecutionTime [$__auto])
by (Path)
```
- query option
	- **Type : Instant**
- Visualization : Bar Cahrt
- Horizontal
- X Axis : Path
- Standard options
	- Unit : milliseconds(ms)

## docker-compose 설정
프로덕션 서버 환경을 기준으로 다음과 같이 docker-compose 파일을 구성하였습니다.
```yaml
version: "3.8"  
services:  
  app:  
    container_name: fineAnts_app  
    build: .  
    restart: always  
    ports:  
      - "443:443"  
    environment:  
      PROFILE: production  
      TZ: Asia/Seoul  
    volumes:  
      - ./logs:/app/logs  
    depends_on:  
      - redis  
    networks:  
      - spring-net  
  redis:  
    container_name: fineAnts_redis  
    image: redis:latest  
    restart: always  
    volumes:  
      - ./redis.conf:/usr/local/etc/redis/redis.conf  
    command: redis-server /usr/local/etc/redis/redis.conf  
    ports:  
      - "6379:6379"  
    networks:  
      - spring-net  
  promtail:  
    image: grafana/promtail:latest  
    container_name: promtail  
    volumes:  
      - ./promtail:/etc/promtail  
      - ./logs:/var/log  
    env_file:  
      - ./secret/promtail/.env  
    command:  
      - -config.file=/etc/promtail/config.yaml  
      - -config.expand-env=true  
networks:  
  spring-net:
```
- volume을 이용하여 설정 파일이 들어있는 promtail 디렉토리를 연결합니다.
- ./logs 디렉토리를 기준으로 볼륨을 통하여 spring의 로그를 promtail이 수집할 수 있도록 합니다.


## Logback Log Rotation 설정
INFO 레벨 로그를 수집하는 파일 appender를 다음과 같이 구현합니다.
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<included>  
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <filter class="ch.qos.logback.classic.filter.LevelFilter">  
            <level>info</level>  
            <onMatch>ACCEPT</onMatch>  
            <onMismatch>DENY</onMismatch>  
        </filter>        
        <file>${LOG_PATH}/info/${INFO_LOG_FILE_NAME}.log</file>  
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">  
            <!-- 이 옵션이 없을 경우 한글이 깨지는 경우 있음-->  
            <charset>${CHARSET}</charset>  
            <pattern>${LOG_PATTERN}</pattern>  
        </encoder>        <!-- Rolling 정책 -->  
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">  
            <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->  
            <fileNamePattern>${INFO_LOG_FILE_PATTERN}</fileNamePattern>  
            <timeBasedFileNamingAndTriggeringPolicy                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">  
                <!-- 파일당 최고 용량 kb, mb, gb -->                <maxFileSize>${MAX_FILE_SIZE}</maxFileSize>  
            </timeBasedFileNamingAndTriggeringPolicy>            <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->  
            <maxHistory>${MAX_HISTORY_DAYS}</maxHistory>  
        </rollingPolicy>    </appender></included>
```
- filter 태그에 의해서 info 레벨 로그만 수집하고 다른 로그 레벨은 수집하지 않습니다.

## Amazon S3 저장소로 내보내기(Export to Amazon S3)
Cloud Log Export 기능을 사용하여 Grafana Cloud Logs에 저장된 로그들을 Amazon S3 저장소에 내보낼 수 잇습니다.

### 사전 작업
시작하기 전에 다음과 같은 사전 작업이 필요합니다.
- Amazon Cloud IAM(Identity and Access Management) Role 설정
- 만약 버전화된 버킷을 생성하는 경우에 다음 권한도 필요합니다.
	- `GetObjectVersion` 권한
	- `GetObjectVersionAttributes` 권한
- 버킷 이름(BUCKET_NAME)
- 버킷 리전(BUCKET_REGION)
- GRAFANA_PRINCIPAL_ARN

### 수행 과정
1. 그라파나에서 **Administration > Plugins and data** 메뉴를 클릭합니다.
![[Pasted image 20250121141123.png]]

2. 다음 화면에서 **Plugins** 메뉴를 클릭합니다.
![[Pasted image 20250121141305.png]]
3. **Cloud logs exporter** 플러그인을 검색합니다.
![[Pasted image 20250121141412.png]]

4. 해당 플러그인을 설치합니다.
![[Pasted image 20250121141434.png]]
5. Cloud logs exporter 화면에서 Amazon S3 버튼을 클릭합니다.
![[Pasted image 20250121141735.png]]
6. Amazon S3 버튼을 클릭하면 AWS S3 권한과 관련된 메뉴얼을 보여줍니다.
![[Pasted image 20250121142635.png]]
![[Pasted image 20250121142720.png]]

7. AWS S3 서비스 -> 버킷 선택 -> 권한 메뉴 이동 -> 버킷 정책의 편집 버튼 클릭
![[Pasted image 20250121144058.png]]

8. 다음과 같이 버킷 정책을 편집합니다. 그리고 변경 사항을 저장합니다.
Resource 구문에서 실제 버킷 이름으로 대체해야 합니다.
![[Pasted image 20250121144444.png]]

9. 다시 Cloud logs exporter 메뉴로 이동하여 Add your bucket details 단계에서 버킷 이름과 버킷 리전을 다음과 같이 입력합니다. 그리고 Test 테스트 버튼을 클릭하여 테스트해봅니다.
![[Pasted image 20250121144907.png]]

10. Submit 버튼을 클릭합니다. 단계를 마무리합니다.
![[Pasted image 20250121145017.png]]



**References**
- https://grafana.com/docs/grafana-cloud/send-data/logs/export/cle-amazon_s3/



