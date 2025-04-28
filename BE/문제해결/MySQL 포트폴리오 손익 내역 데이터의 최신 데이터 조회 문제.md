
## 배경
- 데이터베이스에 포트폴리오 손익 내역 데이터가 300만개인 상태에서 포트폴리오 종목 조회시 응답 시간이 길어지는 문제가 발생하였습니다.
- api : /api/portfolio/:portfolioId/holdings

## 원인
- 비즈니스 로직에서 JPQL을 이용하여 최신 포트폴리오 손익 내역 데이터를 가져오는 경우 300만개를 한꺼번에 가져오고 스트림으로 변환하고 그중 첫번째를 가져오고 있어서 시간이 많이 소모됩니다.
문제의 원인이 되는 코드는 다음과 같습니다.
```java
PortfolioGainHistory latestHistory =  
    repository.findFirstLatestPortfolioGainHistory(portfolio.getId(), LocalDateTime.now(),  
          PageRequest.of(0, 1))  
       .stream()  
       .findFirst()  
       .orElseGet(() -> PortfolioGainHistory.empty(portfolio));
```
