
- [[#The task of collecting logs|The task of collecting logs]]
- [[#Ways to launch Loki|Ways to launch Loki]]
	- [[#Ways to launch Loki#Single-binary|Single-binary]]
- [[#SSD: Simple Scalable Deployment|SSD: Simple Scalable Deployment]]
- [[#Microservices mode|Microservices mode]]
- [[#The arrangement of Grafana Loki architecture|The arrangement of Grafana Loki architecture]]
- [[#Minimum Loki configuration|Minimum Loki configuration]]
	- [[#Minimum Loki configuration#Filesystem|Filesystem]]
- [[#S3 as a storage|S3 as a storage]]
- [[#Cluster and High Availability solutions configuration|Cluster and High Availability solutions configuration]]
- [[#Timeouts|Timeouts]]
- [[#Message sizes|Message sizes]]
- [[#Chunks|Chunks]]
- [[#Parallelism|Parallelism]]
- [[#Optimizing the Write Path|Optimizing the Write Path]]
- [[#Summing up|Summing up]]
- [[#References|References]]


## The task of collecting logs
어떤 로깅 시스템을 통합하기 위해 시도하기 전에 여러분들 스스로 다음 4가지 질문에 답해보세요.
1. 나는 지금 어떻게 로그를 모으고 있나요?
2. 추후에 로그들을 쉽게 처리하기 위해서 로그들에서 올바른 메타데이터를 어떻게 추출하는가?
3. 나는 로그 데이터를 빨리 저장하고 빨리 검색할 수 있도록 어떻게 데이터를 저장하는가?
4. 나는 어떻게 로그를 쿼리하는가?

모든 시스템, 예를 들어 syslog, Elasticsearch, ClickHouse 기반 시스템, 심지어 Grafana Loki 자체도 이 질문들에 대해서는 서로 다르게 응답할 것입니다.

그래서 우리가 아키텍처에 대해서 의논할때, Grafana Loki가 Elasticsearch와 개념적으로 어떻게 다른지, 그리고 로그 저장비용 측면에서 Loki가 왜 좋은지 이야기 할것입니다.

로그 컬렉션 파이프라인은 일반적으로 다음과 같이 간단하고 직관적입니다.
![[Pasted image 20250210152916.png]]

우리는 다양한 데이터 소스들에서 로그들을 얻습니다. 대표적으로 쿠버네티스 클러스터, 가상 머신, 도커 컨테이너 등이 있습니다. 이 데이터 소스들은 로그들을 수집하고, 처리하고 단계를 필터링을 수행합니다.
그리고 나서 데이터 소스들은 여러분들이 지정한 저장소에 데이터들을 저장합니다. 예를 들어 ClickHouse, S3, Grafana Loki와 같은 저장소가 있습니다. 하지만 데이터를 서로 다른 쪽에서 가져오는 모든 사용자가 각기 다른 액션 스크립트를 가질수 있다는 점을 유의해야 합니다.  예를 들어 1년 이나 10분 이내의 데이터를 가져올수 있습니다.

## Ways to launch Loki
Loki를 실행하는 3가지 방법이 있습니다. 이 방법들에는 대체로 규모에서 차이가 있습니다.

### Single-binary
![[Pasted image 20250210153716.png]]

이 방법은 주요한 로키 튜토리얼에서 가장 많이 사용하는 방법입니다. 로직은 다음과 같습니다. 우리는 스토리지와 연결합니다.

스토리지의 역할은 파일 시스템과 S3 버킷 모두에서 수행됩니다. 하지만 여기서는 중요치 않습니다. 이 접근은 장점들을 가지고 있습니다. 예를 들어 런칭이 쉽습니다. 여러분들은 최소한의 설정만 하면 됩니다. 단점은 낮은 **내결함성**입니다. 만약 머신에 장애가 발생하면 로그가 기록되지 않습니다.

위와 같은 상황은 다음과 같이 개선할 수 있습니다.
- 서로 다른 2개의 가상 머신에서 같은 설정을 가진 로키 프로세스를 실행시킵니다.
- memberlist 영역을 사용하여 클러스터로 결합시킵니다.
- 스토리지 전면에 Nginx나 HAProxy와 같은 프록시 서버를 둡니다.
- 가장 중요한 것은 모든것을 하나의 스토리지에 연결하는 것입니다.

여러분들은 프로세스를 추후에 3,4,5개의 노드로 증가시킬 수있습니다.  결과적으로 다음과 같은 결과를 얻을 수 있습니다.
![[Pasted image 20250210154851.png]]

로키 같은 경우에는 쓰기와 읽기 둘다 처리합니다. 따라서 읽기 및 쓰기 처리 요청에 대한 부하가 모든 로키 인스턴스들에 고르게 분배됩니다. 그러나 실제는 고르게 분배되지 않습니다. 왜냐하면 때로는 읽기가 많고, 다른 때는 쓰기가 많기 때문입니다. 

## SSD: Simple Scalable Deployment
두번째 방법은 첫번째 방법을 따르면서 읽기와 쓰기 처리하는 로키 프로세스를 분리하는 것입니다. 예를 들어 디스크에 의존적인 프로세스는 하나의 하드웨어에서 실행하고, 덜 의존적인 프로세스는 다른 하드웨어에서 실행할 수 있습니다. 

여러분들이 로키를 실행할때 -target=write 또는 -target=read 옵션을 전달합니다. 그러면 로키 프로세스들은 특정 쿼리 경로에 책임이 있는 엔티티들을 실행합니다. 간단하게 여러분들은 앞에 프록시 서버가 필요합니다. 프록시 서버가 쓰기 쿼리이면 write 노드에게 서빙하고 읽기 쿼리이면 read 노드에게 서빙합니다.
![[Pasted image 20250210155831.png]]

Grafana는 이를 가장 권장하는 방식으로 간주하고 이를 적극적으로 개발하고 있습니다.

## Microservices mode
마이크로서비스 모드는 우리가 독립적인 각각의 로키 컴포넌트를 실행할 때 사용할 수 있는 방법입니다. 

구성 요소가 많지만, 이들은 쉽게 서로 분리될 수 있으며, 2개 또는 3개의 그룹으로 나눌수 있습니다.
![[Pasted image 20250210160444.png]]
1. 쓰기 컴포넌트 그룹 : Distributor, Ingester
2. 읽기 컴포넌트 그룹 : Querier, Query-frontend, Index-gateway
3. 그외 모든 유틸리티 : Caches, Compactor 등

## The arrangement of Grafana Loki architecture
조금더 아키텍처에 대해서 살펴보며, 각 영역에서 무엇이 설정되었는지 이해해봅니다.

먼저 블록 내 데이터가 어떻게 인덱싱되는지 살펴보겠습니다. Elasticsearch는 전체 텍스트에서 모든 문서들을 기본적으로 인덱싱하는 반면에 Grafana Loki는 로그의 내용을 인덱싱하지 않고 오직 시간이나 레이블과 같은 메타 데이터만 인덱싱합니다. 이 레이블은 프로메테우스 레이블과 매우 비슷합니다. 

결과적으로 Garafana에는 매우 작은 데이터가 포함되어 있기 때문에 매우 작은 데이터 인덱스를 저장하게 됩니다. 여기서 강조하고 싶은 점은 Elasticsearch의 경우 인덱스가 실제 데이터보다 비대해진다는 점입니다.
![[Pasted image 20250210161707.png]]

우리는 인덱싱되지 않은 데이터를 나타나는 순서 그대로 유지합니다. 만약 필터링을 원한다면 우리는 로키에 내장된 grep과 같은 검색 기능을 사용합니다.

### Stream
- **Stream은 로그가 동일한 소스에서 왔어도 고유한 레이블 집합입니다.** 
- Stream은 로그 데이터를 구분하고 조직화하는 기본 단위입니다. 스트림은 로그의 메타데이터와 관련된 정보로, 여러 로그 항목들을 하나로 묶어 관리할 수 있도록 합니다. Loki는 이 스트림을 기반으로 로그를 처리하고 쿼리합니다.

**Stream 구성 요소**
- Stream Labels(라벨)
	- 각 스트림에는 라벨이 있으며, 이는 **로그 데이터의 식별자 역할**을 합니다. 예를 들어, 로그를 **서비스 이름, 호스트, 환경** 등으로 구성할 수 있습니다.
	- 예시: job="api-service", environment="production"
- Log Entries(로그 항목)
	- 각 스트림은 **여러 개의 로그 항목을 포함**할 수 있습니다. 각 로그 항목은 **타임 스탬프, 로그 메시지** 등을 포함합니다.
- Log Streams
	- 스트림은 여러 로그 항목을 순차적으로 가지며, 시간 순대로 정렬됩니다.

**Stream 예시**
Loki에서는 스트림을 기준으로 로그를 수집하고 검색합니다. 예를 들어, 다음과 같은 스트림이 있을 수 있습니다.
```text
stream_1: {job="api-service", environment="production"}
stream_2: {job="frontend-service", environment="staging"}
```
- stream_1에는 API 서비스의 로그가 포함되며, `job="api-service"`와 `environment="production"` 라벨을 가집니다.
- stream_2에는 프론트 엔드 서비스의 로그가 포함되며, `job="frontend-service"`와 `environment="staging"` 라벨을 가집니다.

**Stream의 할용**
1. 구분 및 검색
	- 다양한 서비스나 환경에 대한 로그를 구분하여, 특정 서비스나 환경에 대한 로그를 쉽게 검색할 수 있습니다.
2. 효율적인 집계
	- 스트림 라벨을 기반으로 쿼리할 때, 특정 라벨의 스트림들만 집계하여 분석할 수 있습니다. 예를 들어, `job="api-service"`인 모든 로그를 한번에 조회할 수 있습니다.
3. 멀티테넌시(Multi-tenancy)
	- 스트림을 사용하면 Multi-tenancy 환경에서도 로그 데이터를 격리하고 각 tenant에 맞는 데이터만 조회할 수 있습니다.

**Loki에서 Stream 처리**
- Loki는 데이터를 청크(chunk) 단위로 처리하는데, 각 청크는 하나의 스트림에 해당합니다. 이 스트림은 라벨에 따라 그룹화되며, 로그 항목들은 각 스트림 내에서 시간 순선대로 저장됩니다.
- 쿼리를 할때, 사용자는 스트림 라벨을 기준으로 원하는 로그 데이터를 필터링하고 검색합니다.

**Stream 관련 설정 예시**
```yaml
{job="api-service", environment="production"}
```
이 쿼리는 `job="api-service"`와 `environment="production"` 라벨을 가진 모든 로그 스트림을 조회합니다.

**Stream 정리**
- Stream은 Loki에서 로그 데이터를 구분하고 저장하는 기본 단위로, 라벨을 기반으로 여러 로그 항목을 하나의 스트림으로 묶어서 처리합니다.
- Stream은 쿼리, 필터링, 데이터 격리에 매우 중요한 역할을 하며, 효율적인 로그 관리와 분석에 도움을 줍니다.

**멀티테넌시(Multi-tenancy)**
멀티테넌시 환경은 하나의 시스템, 애플리케이션 또는 서비스 인스턴스가 여러 테넌트(tenant), 즉 사용자 그룹이나 조직에 의해 독립적으로 사용되는 환경을 의미합니다. 각 tenant는 자신의 데이터, 설정, 사용자 등을 격리된 상태로 관리할 수 있습니다. Loki와 같은 시스템에서는 멀티테넌시가 특히 중요하며, 각 테넌트는 로그 데이터를 독립적으로 처리하고 관리할 수 있습니다.

**테넌트(Tenant)**
- 시스템에서 자기만의 데이터, 설정, 사용자 등을 가진 독립적인 사용자 그룹을 의미함.

**Loki에서의 멀티테넌시**
- Loki에서는 멀티테넌시를 통해 여러 팀이나 서비스가 하나의 Loki 인스턴스를 사용하되, 서로의 로그 데이터를 격리할 수 있습니다.
- 각 테넌트는 자신만의 로그 스트림을 관리하며, 데이터를 조회할 때도 자기 테넌트의 데이터만 조회할 수 있습니다.

**Loki에서의 멀티테넌시 예시**
- 테넌트 A : `job="api-service"` , `environment="production"`
- 테넌트 B : `job="frontend-service"`, `environment="staging"`
각 테넌트는 자신만의 로그 스트림을 가지며, 다른 테넌트의 데이터에 접근할 수 없습니다.




![[Pasted image 20250210162014.png]]

이 경우에는 component="suuplier" 레이블은 새로운 스트림을 생성합니다. 이는 앞으로 필요할 것입니다. 속도 제한 및 제약과 관련된 설정이 종종 스트림에 적용되기 때문입니다.
**청크(Chunk)는 여러개의 로그 라인들의 집합입니다.**  로그 라인들을 가져와 하나의 엔티티로 묶고, 이를 Chunk라고 부른 후에 압축하여 스토리지에 저장합니다.

WritePath. Loki에서 로그 입력 지점은 Distributor 엔티티입니다. Distributor 엔티티는 무상태 컴포넌트로, 쿼리를 한개 이상의 Ingester로 분배하는 역할을 수행합니다.

Ingester는 상태를 가지고 있는 컴포넌트입니다. 이들은 일명 해시 링(hash ring)으로 구성하며, 일관된 해싱 시스템입니다. 이는 Ingester를 클러스터로부터 쉽게 추가하거나 제거할 수 잇도록 합니다. 모든 Ingester들은 같은 스토리지에 연결되어 있습니다.

> [!info]
> 해시 링(Hash Ring)은 분산 시스템에서 데이터를 효율적으로 분산하고 노드 추가/제거 시 데이터 재분배를 최소화하기 위한 일관성 해싱(Consistent Hashing) 기법의 핵심 개념입니다.


![[Pasted image 20250210163320.png]]

Readh Path는 복잡하지만 원리는 간단합니다. Query-frontend와 Querier는 무상태 컴포넌트입니다. Query-frontend는 쿼리를 분할하여 더 빠르게 실행할 수 있도록 하는 구성 요소입니다. 
예를 들어 한달의 기간동안의 데이터를 쿼리해야 한다고 가정해봅니다. Query-frontend를 사용하여 쿼리를 더 작은 시간 간격으로 분할하고 이를 Querier에게 병렬로 전송합니다. 그런 다음에 결과들을 병합하고 클라이언트에게 반환합니다.

Querier는 스토리지로부터 로그를 쿼리하는 컴포넌트입니다. 만약 Ingester에서 아직 저장소에 기록되지 않은 새로운 데이터를 받으면, Querier는 그 데이터도 같이 요청합니다.
![[Pasted image 20250210164031.png]]

## Minimum Loki configuration
### Filesystem
Loki가 싱글 인스턴스 모드에서 수행할때 파일 시스템(filesystem)과 함께 작업하는데 중점을 둡니다. 가장 중요한 설정 영역에 대해서 말씀드리겠습니다.
Loki는 구성 작업을 단순화하고 개선하는 방식으로 지속적으로 발전하는 도구입니다. 좋은 변경점 중 하나는 몇 버전 이전에 도입된 `common` 영역입니다. 구성이 반복되는 설정이 여러개 있을 때, `common`을 사용하면 중복없이 결합시킬 수 있습니다. 즉, 이전에는 ingester, querier, distributor 각각에 대해서 ring, stroage 및 다른 요소들을 별도로 설정해야 했지만, 이제는 모든 곳을 common 영역 안에서 공통적으로 설정할 수 있습니다.
```shell
auth_enabled: false  
  
  
server:  
	http_listen_port: 3100  
  
  
common:  
	path_prefix: /tmp/loki  
	storage:  
		filesystem:  
			chunks_directory: /tmp/loki/chunks  
			rules_directory: /tmp/loki/rules  
	replication_factor: 1  
	
	ring:  
		instance_addr: 127.0.0.1  
		kvstore:  
			store: inmemory  
  
schema_config:  
	configs:  
	  - from: 2020-09-07  
		store: boltdb-shipper  
		object_store: filesystem  
		schema: v12  
		index:  
			prefix: loki_index_  
			period: 24h
```
여기에 스토리지가 파일 시스템으로 설정되어 있으며, 청크가 저장되는 위치를 보여줍니다. 여기에서 인덱스, 알림 규칙 등을 저장할 위치도 지정할 수 있습니다.

schema_config는 청크 및 인덱스 데이터가 어떻게 저장될지 설정하는 곳입니다. 일반적으로 크게 변화가 없었지만, 가끔 새로운 버전의 스키마 버전이 등장하기도 합니다. 그래서 최신 개선 사항을 활용할 수 있도록, Loki 스키마를 제때 업데이트할 수 있도록 주기적으로 변경 사항들을 읽는 것을 추천합니다.

여러 로키 인스턴스가 동작하면, 이를 "memberlist" 프로토콜을 사용하여 클러스터로 결합해야 합니다.(etcd나 consul과 같은 서드 파티 시스템을 사용할수도 있습니다.) 그것은 Gossip 프로토콜입니다. 이 프로토콜은 특정 원칙에 따라서 Loki 노드를 자동으로 찾습니다.
![[Pasted image 20250210170203.png]]

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
common:
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: memberlist
schema_config:
  configs:
    - from: 2020-09-07
      store: boltdb-shipper
      object_store: filesystem
      schema: v12
      index:
        prefix: loki_index_
        period: 24h
memberlist:
  join_members:
    - loki:7946
```

클러스터를 자동으로 구성하는 방법은 여러가지가 있으며, 모두 잘 동작합니다. 예를 들어, 여러분들은 memberlist.join_members 설정에서 서로 다른 설정을 할수 있습니다.
- 싱글 호스트 주소
- 주소들의 리스트
- dns+loki.local:7946 - Loki가 A/AAAA DNS 쿼리를 사용하여 호스트 목록을 얻습니다.
- dnssrv+_loki._tcp.loki.local - Loki가 SRV DNS 쿼리를 사용하여 호스트와 포트 정보도 함께 얻어옵니다.
- dnssrvnoa+_loki._tcp.loki.local - A/AAAA 쿼리가 없는 SRV DNS 쿼리

왜 Loki 내의 컴포넌트들끼리 알아야 할까요? Loki 내에는 서로에 대해서 알아야 하는 컴포넌트들이 있기 때문입니다. 예를 들어 Distributor들은 Ingester들에 대해서 알아야 합니다. 그러므로 Ingester들은 같은 ring에 등록됩니다. 그런 후에 Distributor들은 기록 요청을 보낼 Ingester가 어딘지 알게 됩니다. 또다른 예시로 Compactor들은 클러스터안에 싱글 인스턴스에서 작업해야 합니다.

## S3 as a storage
S3는 로키 데이터들을 저장하기 위한 가장 권장되는 방법입니다. 특히 쿠버네티스 클러스터에 배포해야 하는 경우에 권장합니다. 우리가 S3 저장소를 사용할 때 설정은 다음과 같이 변경됩니다.
```yaml
auth_enabled: false

server:
  http_listen_port: 3100  
common:
  path_prefix: /tmp/loki
  storage:
    s3: # The "filesystem" section changes to s3
      s3: https://storage.yandexcloud.net
      bucketnames: loki-logs
      region: ru-central1
      access_key_id:
      secret_access_key:
  replication_factor: 1
  ring:
    kvstore:
      store: memberlist
schema_config:
  configs:
    - from: 2020-09-07
      store: boltdb-shipper
      object_store: filesystem
      schema: v12
      index:
        prefix: loki_index_
        period: 24h
memberlist:
  join_members:
    - loki:7946
```

Tips
- s3://. 대신에 https:// 를 사용하세요. https 프로토콜을 사용하면 암호화 연결을 보장할 수 있습니다.
- 저장소를 분배하여 저장히기 위해서 여러개의 버킷 이름들을 설정할 수 있습니다.
- s3 액세스 키를 설정하기 위하여 ACCESS_KEY_ID와 SECRET_ACCESS_KEY 환경 변수를 사용할 수 있습니다.

![[Pasted image 20250210172110.png]]
저장소를 S3로 변경하고, 엔드포인트, 버킷 이름 및 S3에 적용되는 다른 구성들을 설정합니다.

버킷 이름이 복수형이기 때문에 여러개의 버킷들을 설정할 수 있습니다. 이렇게 하면 Loki가 청크 데이터를 고르구 분배하여 단일 버킷에 대한 부하를 줄입니다. 예를 들어, 호스팅 서비스에서 초당 요청 수(RPS) 제한이 있을 때 이러한 설정이 필요합니다.

## Cluster and High Availability solutions configuration
여러개의 노드에서 로그들을 저장한다고 가정합니다. 그들중 하나는 실패하고 여러분은 해당 노드로부터 데이터를 쿼리할 수 없습니다. 왜냐하면 해당 노드는 장기 저장소에 로그를 작성할 시간이 없었기 때문입니다. 
Loki에서 고가용성은 `replication_factor` 옵션을 사용하여 제공합니다. 이 설정에서는 Distributor가 로깅 쿼리를 여러개의 Ingester 복제본에게 전송합니다. 
```yaml
auth_enabled: false
server:
  http_listen_port: 3100
common:
  path_prefix: /tmp/loki
  storage:
    s3:
      s3: https://storage.yandexcloud.net
      bucketnames: loki-logs
      region: ru-central1
      access_key_id:
      secret_access_key:
  replication_factor: 3  # Take a note of this field
  ring:
    kvstore:
      store: memberlist
schema_config:
  configs:
    - from: 2020-09-07
      store: boltdb-shipper
      object_store: filesystem
      schema: v12
      index:
        prefix: loki_index_
        period: 24h
memberlist:
  join_members:
    - loki:7946
```

replication_factor:
- Distributor가 여러개의 Ingester에게 청크 데이터를 전송합니다.
- 3개의 노드를 사용할때 최소 3개의 복제본이 되어야 합니다.
- 3개의 노드중 1개의 실패만 허용합니다.
- maxfailure = (replication_factor / 2) + 1
	- 예를 들어 replication_factor=3인 경우 maxfailture는 2가 됩니다. 즉, 데이터의 1개의 복제본만 실패해도 시스템이 정상 가동합니다.

Distributor는 여러개의 Ingester들에게 한번에 청크 데이터를 전송합니다.
![[Pasted image 20250210173620.png]]

데이터 중복 제거를 위해서 boltdb-shipper는 여러 캐시를 사용합니다. 청크 데이터에 대해서는 `chunk_cache_config` 설정, 인덱스 데이터에 대해서는 `write_dedupe_cache_config` 설정을 사용합니다. 또한 Querier는 중복된 로그를 필터링하는 역할을 수행합니다. 

## Timeouts
Loki를 잘못 설정하면 502나 504와 같은 오류가 자주 발생할 수 있습니다. 이러한 오류들은 주로 네트워크 문제나 시스템 자원 부족, 서버간 통신 문제 등으로 발생할 수 있습니다. 이러한 문제를 피하기 위해서는 Loki 구성 요소들간에 연결과 자원 사용량 등을 면밀히 모니터링하고 설정을 점검하는 것이 중요합니다.
![[Pasted image 20250210174437.png]]

오류를 더 잘 이해하기 위해서는 먼저 프로젝트에서 타임아웃 값을 충분히 늘려야 합니다. 두번째로, 여러 유형의 타임아웃을 적절하게 구성해야 합니다.
4. http_server_{write,read}_timeout 설정은 웹 서버 응답 시간에 대한 기본적인 타임아웃을 설정합니다.
5. querier.query_timeout과 querier.engine.timeout 설정은 직접적으로 읽기 쿼리를 실행하는 엔진의 최대실행 시간을 설정합니다.
```yaml
server:
  http_listen_port: 3100
  http_server_write_timeout: 310s
  http_server_read_timeout: 310s
querier:
  query_timeout: 300s
  engine:
    timeout: 300s

```

만약 Loki의 앞단에 Nginx와 같은 적절한 프록시 서버를 사용하고 있다면, 여러분들은 그 nginx 서버 또한 타임아웃 값을 증가시켜야 합니다.
```
server {  
	proxy_read_timeout = 310s;  
	proxy_send_timeout = 310s;  
}
```

여러분들은 그라파나 사이드에도 타임아웃을 설정해야 함니다. 
```
[dataproxy]  
timeout = 310
```

가장 좋은 접근 방법은 querier에서 모든 4가지 유형의 타임아웃을 최솟값으로 설정하는 것입니다.(예: 300초) 그렇게 하면 쿼리가 먼저 완료되고, 그 다음에 HTTP 서버, Nginx 또는 그라파나와 같은 다른 서비스들이 조금 더 오래 걸리게 됩니다.
기본적으로 타임아웃은 매우 작습니다. 그래서 나는 이 값을 증가시키는 것을 권장합니다.

## Message sizes
일부 오류의 원인이 명확하지 않아서 이 주제가 복잡하게 느껴질 수 있습니다.

Message sizes 속성을 나타내는 grpc_server_max_{recv,send}_msg_size 설정은 로그 사이즈의 최대 크기를 의미합니다. 기본적인 최대 크기는 매우 작습니다. 예를 들어 큰 스택 트레이스가 있고 20MB의 로그가 하나의 로그 라인으로 전송된다면, 해당 제한을 맞출수 있는 방법은 없습니다. 따라서 이 제한을 늘려야 합니다.
```yaml
server:  
	http_listen_port: 3100  
	grpc_server_max_recv_msg_size: 104857600 # 100 Mb  
	grpc_server_max_send_msg_size: 104857600 # 100 Mb  
  
  
ingester_client:  
	grpc_client_config:  
		max_recv_msg_size: 104857600 # 100 Mb  
		max_send_msg_size: 104857600 # 100 Mb
```
- 기본 로그 송신/수신 최대 사이즈는 4MB입니다.
- Loki가 처리하는 로그의 사이즈에 직접적인 영향을 미칩니다.

우리는 청크 데이터의 인코딩 또한 피할 수 없습니다. 기본 값은 최대로 압축하는 gzip 확장자 파일입니다. 그라파나는 snappy로 변경하는 것을 권장하고 경험을 기반으로 snappy를 사용하는 것에 동의하고 있습니다. 그리고 나서 로그는 저장소에서 조금 더 많은 공간을 가지지만 읽기 및 쓰기 데이터는 더 효율적입니다.

> [!info]
> snappy는 Google에서 개발한 고성능 데이터 압축 알고리즘입니다. 주로 빠른 압축 및 압축 해제 속도를 목표로 설계되었으며, 데이터 압축률보다는 처리 속도에 중점을 두고 있습니다.


```yaml
ingester:  
	chunk_encoding: snappy
```

기본적인 인코딩은 gzip입니다.
- 더 나은 압축률
- 느린 쿼리 속도
우리는 인코딩하는데 snappy를 권장합니다.
- 압축률이 조금 더 낮음
- 그러나 인코딩/디코딩하는데 빠름

## Chunks
청크의 크기와 수명에 관하여 많은 설정이 있습니다. 저는 너무 많은 설정을 하는 것을 권장하지 않습니다. 그리고 여러분들은 여러분들이 이 설정 값을 변경하였을때 어떤 일이 발생하는지 정말로 이해해야 합니다. 
```yaml
ingester:  
	chunk_idle_period: 2h  
	chunk_target_size: 1536000  
	max_chunk_age: 2h
```
- 저는 `chunk_block_size`과 `chunk_retain_period` 를 변경하는 것을 권장하지 않고 기본 값을 사용하는 것을 권장합니다.
- 청크 데이터가 대부분 가득차 있는 경우 `chunk_target_size` 를 늘릴 수 있습니다. 이 값을 설정하면 청크에 더 많은 공간을 확보할 수 있습니다.
- `chunk_idle_period` 는 청크에 기록이 전혀 없는 경우 해당 청크가 Ingester의 메모리에서 얼마나 오래 살아있을지 설정하는 값입니다. 따라서 스트림이 대부분 느리고 반쯤 비어있는 경우에는 이 기간을 늘리는 것이 좋습니다. 기본값은 30분입니다.

## Parallelism
```yaml
querier:  
	max_concurrent: 8  
  
  
limits_config:  
	max_query_parallelism: 24  
	split_queries_by_interval: 15m  
  
  
frontend_worker:  
	match_max_concurrent: true
```
- querier.max_concurrent : 하나의 Querier가 병렬로 처리할 수 있는 쿼리의 최대 개수 설정값입니다. 저는 CPU 개수의 2배로 설정하는 것을 권장합니다. 기본값은 10입니다.
- limits_config.max_query_parallelism : tenant가 가질수 있는 최대 병렬 쿼리 처리 개수를 설정하는 값입니다. `querier.max_concurrent` 값은 `max_query_parallelism` 과 다음 공식에 따라 일치해야 합니다.
	- querier count * max_concurrent >= max_query_parallelism
	- 쿼리 개수 * 최대 병렬성의 값이 최대 쿼리 병렬 처리 개수보다 이상이어야 합니다.

우리의 예시에서는 병렬 처리 값 24를 제공하기 위해서는 **최소 3개의 Querier**가 실행되어야 합니다.

## Optimizing the Write Path
여기에 쓰기 관련된 설정이 몇가지 있습니다.
- ingestion_write_mb
	- tenant가 초당 수집할 수 있는 최대 로그 데이터 크기(메가바이트 단위)를 설정합니다.
	- 예를 들어 ingestion_rate_mb=20 이면 초당 최대 20MB의 로그 데이터만 수집할 수 있도록 제한합니다.
- ingestion_burst_size_mb
	- tenant가 버스트 모드에서 초과할 수 있는 최대 데이터 크기(메가 바이트 단위)를 설정합니다.
	- 예를 들어 ingestion_burst_size_mb=30 이면 초당 최대 수집 데이터가 30MB만 수집할 수 있도록 제한합니다.

```yaml
limits_config:  
	ingestion_rate_mb: 20  
	ingestion_burst_size_mb: 30
```

기본값은 매우 낮습니다. 그래서 나는 이 설정값들을 설정해서 증가시키는 것을 권장합니다. 

```
limits_config:  
	per_stream_rate_limit: "3MB"  
	per_stream_rate_limit_burst: "10MB"
```
- per_stream_rate_limit : 각 스트림에 대해 초당 허용되는 최대 데이터 전송 속도값 설정.
	- 예를 들어 per_stream_rate_limit=3MB 이면 각 스트림에 대해 초당 최대 3MB의 로그만 처리할 수 있습니다.
	- 개별 로그 스트림에 대한 속도 제한을 두어서 특정 스트림이 과도한 데이터를 전송하는 것을 방지합니다.
- per_stream_rate_limit_burst : 버스트 모드에서 초과할 수 잇는 최대 데이터 크기입니다.
	- 예를 들어 per_stream_rate_limit_burst=10MB 이면 초당 3MB로 제한되지만, 일시적으로 10MB까지의 데이터를 처리할 수 있습니다. 이 값은 급증하는 트래픽에 대응할 수 있도록 합니다.

> [!info]
> tenant는 로그 데이터를 격리하고 관리할 수 있는 논리적 단위입니다. tenant는 다중 사용자 환경에서 서로 다른 사용자나 팀의 로그 데이터를 분리하고 독립적으로 처리하기 위해 사용됩니다.

## Summing up
- 로그 수집 작업
- 로키 아키텍처
- 로키 런칭 전략의 3가지 유형
	- Single-Binary
	- Simple Scalable Deployment
	- Mircoservice mode
- replication_factor 설정을 한 로키의 고 가용성
- 타임아웃과 메시지 크기의 설정
- 병렬성
- write path를 최적화하는 것은 일반적으로 어려운 처리가 아니다. 모니터링 지표를 살펴보고 그에 따라 최적화하면 됩니다.

## References
- https://medium.com/lonto-digital-services-integrator/grafana-loki-configuration-nuances-2e9b94da4ac1