
## 배경
Spring 서버가 스케일 아웃하면서 동시 실행하는 구조가 되면서 각 인스턴스에 등록된 `@Scheduled` 작업이 중복 실행되는 문제가 발생할 수 있습니다. 예를 들어 5초마다 종목의 현재가를 갱신하는 스케줄러 작업이 있습니다. 해당 작업은 외부 API에 요청하여 종목의 현재가를 질의한 다음에 Redis에 저장합니다. 종목의 현재가 데이터는 여러 Spring 인스턴스가 공통적으로 사용하기 때문에 각각의 Spring 인스턴스가 스케줄러 작업을 수행 할 필요없이 하나의 Spring 인스턴스가 맡아서 수행하여 데이터를 저장하면 됩니다. 하지만 `@Schedule`

### 문제정의
- 다중 인스턴스 환경에서 `@Scheduled` 메서드가 모든 인스턴스에서 실행됩니다.
- 한번만 실행되어야 할 작업이 중복 실행됩니다.
- 리소스 낭비, 장애 유발, API 호출 제한 등이 유발됩니다.

### 해결 방안 제안
1. Redis 기반 분산 락
2. ShedLock 라이브러리 활용
3. 특정 인스턴스 전용 프로파일로 실행

| 구분          | Redis 기반 분산 락                                                                                                                                          | ShedLock 라이브러리 활용                                                                                   | 특정 인스턴스 전용 프로파일로 실행                                                                                                                               |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 장점          | - 쉽고 실용적<br>- 이미 Redis를 사용하는 환경에 적합                                                                                                                    | - 설정이 간단하고 Spring 환경에 최적화되어 있습니다.  <br>- 저장소(RDB, Redis, Mongo 등)에 유연성을 가짐                          | - 설정이 간단<br>- 안정성이 높음                                                                                                                             |
| 단점         | - Redis 장애시 전체 시스템에 영향이 간다<br>- 네트워크 지연에 민감. 클러스터 환경에서 Redis와의 통신 지연이 문제 될 수 있음<br>- TTL 설정 실수 시 데드락 또는 중복 실행 발생 가능성이 있음<br>- Redisson 사용시 복잡한 설정이 필요함 | - 락 갱신 주기에 민감함<br>- DB에 락 테이블이 필요함<br>- 모든 Job에 명시적으로 애노테이션 추가 필요함<br>- 스케줄러 간 충돌 방지용 버퍼 설정 고려가 필요함 | - 자동 failover 불가능<br>- 수동 관리 필요함. 배포 시 해당 인스턴스 구분 및 설정이 필요함<br>- 스케줄러 인스턴스가 중복 배포될 경우 제어 불가능함<br>- 배포 관리자 실수로 여러 인스턴스가 `schduler` 프로파일을 가질 위험이 있음 |
| 핵심 기술       | Redis(setnx, Redisson)                                                                                                                                 | DB/Redis/Mongo                                                                                      | Spring Profile                                                                                                                                    |
| 락 있음        | O                                                                                                                                                      | O                                                                                                   | X                                                                                                                                                 |
| 자동 failover | O                                                                                                                                                      | O                                                                                                   | X                                                                                                                                                 |
| 적합 환경       | Redis 운영중인 환경                                                                                                                                          | 간단한 분산 환경                                                                                           | 단순 운영 환경                                                                                                                                          |

## 선택한 전략 : ShedLock 라이브러리 활용
### 선택 이유
- 현재 서버 아키텍처 구조가 Spring 서버와 Redis를 사용하고 있습니다.
- ShedLock 라이브러리가 Redis 저장소를 지원합니다.
- 특정 인스턴스 전용 프로파일로 실행하는 방법은 자동 failover가 불가능하고, 수동 관리가 필요하기 때문에 선택하지 않았습니다.
- Redis 기반 분산 락 방법 대신 선택한 이유
	- ShedLock 라이브러리가 상대적으로 구현 난이도가 간단합니다. 반면 Redis 기반 분산 락 방법은 직접 락을 구현해야 하거나 Redisson에 의존해야 합니다.
	- Spring 통합성 부분이 ShedLock 라이브러리가 더 높기 때문입니다. ShedLock 라이브러리 사용시 `spring Schedule + @Scheduler + @EnableScheduling` 조합으로 바로 사용이 가능합니다. 반면 Redis 기반 분산 락 방법은 별도 락 흭득 로직이 필요합니다.
	- ShedLock 라이브러리를 활용하면 Redis만이 아닌 다른 RDB(MySQL, MongoDB)로 변경하기 쉽습니다. 또한 RDB 기반일 경우 트랜잭션 처리 연계가 가능합니다.
	- ShedLock 라이브러리는 스케줄러 락 전용 라이브러리이지만 Redis 기반 분산 락 방법은 범용 분산 락이기 때문에 부가적인 설정이 필요합니다.
- ShedLock 사용시 MySQL이 아닌 Redis를 선택한 이유
	- Redis를 이미 서비스에서 운영중이기 때문
	- 종목의 현재가 갱신이 5초에 한번씩으로 짧은 주기로 설정되어 있음. 이는 락 처리 성능이 중요하기 때문에 Redis를 선택
	- Redis 사용시 TTL 위주 자동락 해제, 빠른 락 처리를 지원함


## ShedLock 라이브러리 적용
### 의존성 추가
```gradle
implementation 'net.javacrumbs.shedlock:shedlock-spring:6.3.1'  
implementation 'net.javacrumbs.shedlock:shedlock-provider-redis-spring:6.3.1'
```
- 6.3.1 버전은 현재 작성 시점에 최신 버전입니다.
- sheldlock-spring:6.3.1은 Spring 6, Spring Boot 3과 호환됩니다.
- Spring Boot 3 이상의 프로젝트를 운영중이면 6.x 이상의 ShedLock 버전이 필수적입니다.

### 스케줄러 설정 클래스 구현
```java
@EnableSchedulerLock(defaultLockAtLeastFor = "1m", defaultLockAtMostFor = "1m")  
@EnableScheduling  
@ConditionalOnProperty(value = "scheduling.enabled", havingValue = "true", matchIfMissing = true)  
@Configuration  
public class SchedulerConfig {  
  
    private final Environment env;  
  
    public SchedulerConfig(Environment env) {  
       this.env = env;  
    }  
  
    @Bean  
    public LockProvider lockProvider(RedisConnectionFactory connectionFactory) {  
       String lockEnv = env.getProperty("spring.profiles.active", "default");  
       return new RedisLockProvider(connectionFactory, lockEnv);  
    }  
}
```
- 락 정보를 Redis에 저장하기 위해서 RedisLockProvider 스프링 빈을 설정합니다.


### 스케줄러 설정
```java
  
/**  
 * 3시 30분에 한국투자증권의 모든 종목의 종가를 갱신합니다.  
 * <p>  
 * 한국투자증권의 모든 종목의 종가를 갱신합니다.  
 * </p>  
 */  
@SchedulerLock(name = "kisClosingPriceScheduler", lockAtLeastFor = "1m", lockAtMostFor = "1m")  
@Scheduled(cron = "${cron.expression.closing-price:0 30 15 * * ?}")  
@Transactional(readOnly = true)  
public void scheduledRefreshAllClosingPrice() {  
    if (fileHolidayRepository.isHoliday(LocalDate.now())) {  
       return;  
    }  
    kisService.refreshAllClosingPrice();  
}
```
- `@SchedulerLock` 애노테이션을 `@Scheduled` 애노테이션이 정의되어 있는 메서드에 선언하여 락 설정을 합니다.
- lockAtLeastFor : 스케줄러 실행이 완료된 후에도 최소 1분은 락을 가지며 유지합니다. 해당 옵션을 설정하면 1분 동안 다른 Spring 인스턴스가 스케줄러에 접근하지 못하고 실행하지 않습니다.
- lockAtMostFor : 스케줄러 메서드가 오류가 발생해서 실패하거나 멈췄을 경우, 이 시간이 지나면 자동으로 락을 해제합니다. 위 예제같은 경우에는 1분이 지나면 자동으로 해제합니다.
	- 옵션을 사용하는 이유는 서버가 죽거나 작업중 예외가 발생해도 락이 그대로 남아버려서 락이 영원히 걸려버립니다. 이 문제를 해결하기 위해서 해당 옵션을 사용해서 자동으로 락을 해제합니다.

### 실행 결과 확인
스케줄러에 락이 걸리는지 확인하기 위해서 Spring 인스턴스 2개를 실행한 다음에 확인해보겠습니다. 그전에 다음과 같이 임시 스케줄러 메서드를 구현합니다.
```java
@Slf4j  
@Component  
public class ExampleScheduler {  
  
    @SchedulerLock(  
       name = "exampleLock",  
       lockAtLeastFor = "10s", // 최소 10초 유지  
       lockAtMostFor = "1m"    // 최대 1분 뒤 자동 해제  
    )  
    @Scheduled(cron = "*/5 * * * * *") // 5초마다 실행  
    public void run() {  
       log.info("🔥 exampleLock 스케줄러 실행됨");  
    }  
}
```

Spring 인스턴스 2개를 실행해서 로그를 각각 확인해봅니다.
다음 실행 결과는 첫번째 Spring 인스턴스의 로그입니다.
![[Pasted image 20250411122548.png]]

두번째 Spring 인스턴스의 로그는 다음과 같습니다.
![[Pasted image 20250411122630.png]]

두번째 Spring 인스턴스의 실행 결과에서 첫번째 로깅의 시각은 2025-04-11 12:25:30에 실행된 것을 볼수 있습니다. 첫번째 Spring 인스턴스의 실행 결과에서 해당 시각을 찾아보면 12:25:20 ~ 12:25:40 사이에 없는 것을 볼수 있습니다. 이는 12:25:30 시각에 두번째 Spring 인스턴스가 스케줄러를 실행하였고 그 시각에 첫번째 Spring 인스턴스는 락이 걸려서 실행하지 못한 것을 알 수 있습니다.

위 실행 결과로 알 수 있는 사실은 2개의 Spring 인스턴스가 동시에 스케줄러를 실행하지 않고 둘 중 하나가 락을 얻어서 실행하는 것을 알수 있습니다. 락을 얻지 못한 다른 Spring 인스턴스는 해당 스케줄러를 실행하지 않고 다른 작업을 수행합니다.

