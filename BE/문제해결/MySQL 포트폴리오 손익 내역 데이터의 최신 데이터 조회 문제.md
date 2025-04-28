
## 배경
- 데이터베이스에 포트폴리오 손익 내역 데이터가 300만개인 상태에서 포트폴리오 종목 조회시 응답 시간이 길어지는 문제가 발생하였습니다.
- api : /api/portfolio/:portfolioId/holdings

## 원인
비즈니스 로직에서 JPQL을 이용하여 최신 포트폴리오 손익 내역 데이터를 가져올 때, 데이터가 300만건에 달하는데 이를 한번에 가져와서 스트림으로 변환하고 그중 첫번째 항목을 추출하는 방식입니다. 이 과정에서 불필요하게 많은 데이터를 메모리로 로드한 후 첫번째 항목을 찾는 방식이기 때문에 성능 문제가 발생하고 있습니다.
```java
PortfolioGainHistory latestHistory =  
    repository.findFirstLatestPortfolioGainHistory(portfolio.getId(), LocalDateTime.now())  
       .stream()  
       .findFirst()  
       .orElseGet(() -> PortfolioGainHistory.empty(portfolio));
```
- MySQL 클라이언트를 이용하여 300만건의 데이터를 가져올때 약 3.5초 정도가 소요되고 스트림 변환 및 첫번째 항목을 가져오는 부분에서 측정할 수 없을 정도로 긴 시간이 소요되었습니다.

## 해결 방법


