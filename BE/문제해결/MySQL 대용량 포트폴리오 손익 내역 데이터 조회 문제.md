
## 배경
- 포트폴리오 손익 내역(PortfolioGainHistory) 데이터가 테이블에 300만개가 존재합니다. 한 포트폴리오에 300만개의 손익 내역이 있습니다.
- 포트폴리오 손익 내역 관련된 API 조회시 처리되지 않고 스택오버플로우가 발생합니다.
- 단, MySQL 클라이언트를 이용하여 조회시 3.2초가 소요됩니다.

## 원인
포트폴리오 손익 내역 데이터(300만개)를 쿼리할 때 115,610ms(115초)가 소요됩니다. 다음 코드에서 portfolioGainHistoryRepository.findAllByPortfolioId 메서드 호출시 많은 시간이 소요됩니다.
```java
@Transactional(readOnly = true)  
@Secured("ROLE_USER")  
@Cacheable(value = "lineChartCache", key = "#memberId")  
public List<DashboardLineChartResponse> getLineChart(Long memberId) {  
    List<PortfolioGainHistory> histories = portfolioRepository.findAllByMemberId(memberId).stream()  
       .map(Portfolio::getId)  
       .map(portfolioGainHistoryRepository::findAllByPortfolioId)  
       .flatMap(Collection::stream)  
       .toList();  
  
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

Spring JPA는 조회된 결과를 PortfolioGainHistory 엔티티로 매핑하고, 모든 데이터를 영속성 컨텍스트에 올립니다. 300만개의 엔티티를 메모리에 로드하고 관리하려다 보니 GC 및 메모리 부하가 큽니다.
