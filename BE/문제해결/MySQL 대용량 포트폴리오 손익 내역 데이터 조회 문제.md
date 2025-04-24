
## 배경
- 포트폴리오 손익 내역(PortfolioGainHistory) 데이터가 테이블에 300만개가 존재합니다. 한 포트폴리오에 300만개의 손익 내역이 있습니다.
- 포트폴리오 손익 내역 관련된 API 조회시 처리되지 않고 스택오버플로우가 발생합니다.
- 단, MySQL 클라이언트를 이용하여 조회시 3.2초가 소요됩니다.

## 원인
포트폴리오 손익 내역 데이터(300만개)를 쿼리할 때 115,745ms(115초)가 소요됩니다. 다음 코드에서 portfolioGainHistoryRepository.findAllByPortfolioId 메서드 호출시 많은 시간이 소요됩니다.
```java
@Transactional(readOnly = true)  
@Secured("ROLE_USER")  
@Cacheable(value = "lineChartCache", key = "#memberId")  
public List<DashboardLineChartResponse> getLineChart(Long memberId) {  
    long startTime = System.currentTimeMillis();  
    List<PortfolioGainHistory> histories = portfolioRepository.findAllByMemberId(memberId).stream()  
       .map(Portfolio::getId)  
       .map(portfolioGainHistoryRepository::findAllByPortfolioId)  
       .flatMap(Collection::stream)  
       .toList();  
    long endTime = System.currentTimeMillis();  
    log.debug("executed time: {}ms", endTime - startTime);  
  
    Map<String, Expression> result = histories.stream()  
       .collect(Collectors.toMap(  
          PortfolioGainHistory::getLineChartKey,  
          PortfolioGainHistory::calculateTotalPortfolioValue,  
          Expression::plus  
       ));  
    return result.keySet()  
       .stream()  
       .sorted()  
       .map(key -> DashboardLineChartResponse.of(key, result.get(key)))  
       .toList();  
}
```

findAllByPortfolioId 메서드 실행시 쿼리는 다음과 같습니다.
```java
@Query("select p from PortfolioGainHistory p where p.portfolio.id = :portfolioId")  
List<PortfolioGainHistory> findAllByPortfolioId(@Param("portfolioId") Long portfolioId);
```

위 메서드에서 300만개의 데이터를 조회하는데 115초가 소요되었음에도 다음 코드 실행시 StackOverFlow가 발생합니다.
```java
return result.keySet()  
    .stream()  
    .sorted()  
    .map(key -> DashboardLineChartResponse.of(key, result.get(key)))  
    .toList();
```

정리하면 다음과 같은 문제점을 가지고 있습니다.
- Spring JPA는 조회된 결과를 PortfolioGainHistory 엔티티로 매핑하고, 모든 데이터를 영속성 컨텍스트에 올립니다. 300만개의 엔티티를 메모리에 로드하고 관리하려다 보니 GC 및 메모리 부하가 큽니다.
- 데이터 조회시 시간이 많이 소요(115초)됩니다.
- 300만개의 데이터를 한꺼번에 처리하려니 메모리가 부족하여 StackOverFlow가 발생합니다.

## 해결 방법 탐색
### 포트폴리오 손익 내역 데이터 처리 문제 탐색
포트폴리오 손인내역 데이터를 조회하는데 115초가 소요되기 때문에 이 속도를 감소시키기 위해서 반환타입을 Stream으로 변경해봅니다. 반화타입으로 Stream으로 사용하면 JPA는 결과 전체를 메모리에 로딩하지 않고, 커서를 이용해 순차적으로 한 행씩 읽는 방식을 사용합니다. 이 덕분에 메모리 사용량을 줄이고, GC 오버헤드도 방지합니다. 이 설명을 반영한 코드는 다음과 같습니다.
```java
@Transactional(readOnly = true)  
@Secured("ROLE_USER")  
@Cacheable(value = "lineChartCache", key = "#memberId")  
public List<DashboardLineChartResponse> getLineChart(Long memberId) {  
    List<Long> portfolioIds = portfolioRepository.findAllByMemberId(memberId).stream()  
       .map(Portfolio::getId)  
       .toList();  
  
    Map<String, Expression> result = new HashMap<>();  
    long startTime = System.currentTimeMillis();  
    for (Long portfolioId : portfolioIds) {  
       portfolioGainHistoryRepository.streamAllByPortfolioId(portfolioId)  
          .forEach(history -> {  
             String key = history.getLineChartKey();  
             Expression value = history.calculateTotalPortfolioValue();  
             result.merge(key, value, Expression::plus);  
             entityManager.detach(history);  
          });  
    }  
    long endTime = System.currentTimeMillis();  
    log.debug("executed time: {}ms", endTime - startTime);  
  
    return result.keySet()  
       .stream()  
       .sorted()  
       .map(key -> DashboardLineChartResponse.of(key, result.get(key)))  
       .toList();  
}
```

위 코드를 실행한 결과는 다음과 같습니다. 실행 결과를 보면 115초에서 101초로 감소하기는 했지만 만족스러운 결과는 아닙니다.
```shell
executed time: 101261ms
```

이번에는 프로파일링하여 forEach문 내부에 걸리는 시간을 분석해보겠습니다. 결과는 다음과 같습니다.
![[Pasted image 20250424141906.png]]
위 결과를 보면 entityManager가 detach 메서드를 호출하여 엔티티를 영속성 컨텍스트에서 분리하는 과정이 3,793ms가 소요된것을 볼 수 있습니다.

위 문제를 해결하기 위해서 다음과 같이 변경하여 측정해봅니다. entityManager가 detach 메서드를 호출하는 것이 아닌 1000번째마다 clear() 메서드를 호출해봅니다.
```java
@Transactional(readOnly = true)  
@Secured("ROLE_USER")  
@Cacheable(value = "lineChartCache", key = "#memberId")  
public List<DashboardLineChartResponse> getLineChart(Long memberId) {  
    List<Long> portfolioIds = portfolioRepository.findAllByMemberId(memberId).stream()  
       .map(Portfolio::getId)  
       .toList();  
  
    Map<String, Expression> result = new HashMap<>();  
    int batchSize = 1000;  
    int count = 0;  
    long startTime = System.currentTimeMillis();  
    for (Long portfolioId : portfolioIds) {  
       try (Stream<PortfolioGainHistory> stream = portfolioGainHistoryRepository.streamAllByPortfolioId(  
          portfolioId)) {  
          Iterator<PortfolioGainHistory> iterator = stream.iterator();  
          while (iterator.hasNext()) {  
             PortfolioGainHistory history = iterator.next();  
             String key = history.getLineChartKey();  
             Expression value = history.calculateTotalPortfolioValue();  
             result.merge(key, value, Expression::plus);  
  
             count++;  
             if (count % batchSize == 0) {  
                entityManager.clear();  
             }  
          }  
       } catch (Exception e) {  
          log.error("Error occurred while streaming PortfolioGainHistory for portfolioId: {}", portfolioId, e);  
          throw new IllegalStateException("Error occurred while streaming PortfolioGainHistory", e);  
       }  
    }  
    long endTime = System.currentTimeMillis();  
    log.debug("executed time: {}ms", endTime - startTime);  
  
    return result.keySet()  
       .stream()  
       .sorted()  
       .map(key -> DashboardLineChartResponse.of(key, result.get(key)))  
       .toList();  
}
```

실행 결과는 다음과 같습니다. 실행 결과를 보면 101초에서 88초로 감소되었지만 여전히 만족스러운 수치는 아니었습니다.
```shell
executed time: 88450ms
```

프로파일링 결과는 다음과 같습니다.
![[Pasted image 20250424142716.png]]


### 응답 결과 처리 해결 탐색
포트폴리오 손익 내역 데이터를 이용하여 계사된 해시맵을 이용하여 리스폰스 객체를 생성합니다.
```java
return result.keySet()  
    .stream()  
    .sorted()  
    .map(key -> DashboardLineChartResponse.of(key, result.get(key)))  
    .toList();
```

실행 결과는 다음과 같습니다. 다음 실행 결과를 보면 Bank.reduce() -> Sum.reduce() -> Bank.reduce() 순으로 반복적으로 호출됨을 알수 있습니다.
```shell
java.lang.StackOverflowError: null
	at co.fineants.api.domain.common.money.Bank.reduce(Bank.java:22)
	at co.fineants.api.domain.common.money.Sum.reduce(Sum.java:20)
	at co.fineants.api.domain.common.money.Bank.reduce(Bank.java:22)
	at co.fineants.api.domain.common.money.Sum.reduce(Sum.java:20)
	at co.fineants.api.domain.common.money.Bank.reduce(Bank.java:22)
```

원인은 Expression 타입의 객체가 reduce(Bank, Currency) 메서드를 호출할 때 내부적으로 또다른 Expression이 들어있고 이 구조가 종료 조건없이 계속 호출되면서 StackOverFlow가 발생합니다.

다음 디버깅 화면을 보면 Expression 타입의 실제 구현체 타입은 Sum 타입임을 알수 있습니다.
![[Pasted image 20250424144549.png]]

Sum.reduce() 메서드로 들어가서 다시 augend와 addend가 전부 Sum 타입임을 알수 있습니다. 이러한 합계가 300만개 정도 있는 것을 볼수 있습니다. 그래서 Bank -> Sum -> Bank -> Sum 타입순으로 계속 호출되다가 메모리에 더이상 공간이 없어서 Stackoverflow을 발생시킨 것입니다.
![[Pasted image 20250424144633.png]]

