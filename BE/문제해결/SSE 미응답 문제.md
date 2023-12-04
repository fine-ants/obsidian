## 상황
dev 브랜치 소스 코드를 기준으로 로컬 개발 서버에서 수행할때는 포트폴리오 상세 조회 API 요청시 5초에 한번씩 잘 응답되었지만 이 브랜치를 배포환경에서 실행했을때는 계속 blocking하다가 타임아웃이 발생합니다.

이러한 상황은 UI, 포스트맨, 터미널에서 동일하게 발생하였습니다.

### 원인
- SSE 응답을 Runnable 객체로 만들어 ScheduledExecutorService 쓰레드풀에 스케줄링 하였을때 예외가 발생하였지만 쓰레드여서 발견되지 않았던 것이었습니다. 그리고 IOException과 FineAntsException 예외 종류는 캐치할 수는 있었지만 발생한 예외는 IllegalArgumentException 이었습니다.
- IllegalArgumentException 예외가 발생한 곳은 PortfolioStockService.readMyPortfolioStocks() 메소드 안에 LastDayClosingPriceManager 객체가 수행하는 getPrice 메소드 부분이었습니다. 흐름상 포트폴리오 종목들을 순회하며 티커심볼별 종가 가격 맵을 생성하는 부분이었습니다.
```
Map<String, Long> lastDayClosingPriceMap = portfolioHoldings.parallelStream()  
    .map(PortfolioHolding::getStock)  
    .map(Stock::getTickerSymbol)  
    .collect(Collectors.toMap(key -> key, manager::getPrice));
```

- 그중에서 mnager::getPrice 부분은 다음과 같습니다.
```
public Long getPrice(String tickerSymbol) {  
    String price = redisTemplate.opsForValue().get(String.format(format, tickerSymbol));  
    if (price == null) {  
       throw new IllegalArgumentException(String.format("%s 종목에 대한 가격을 찾을 수 없습니다.", tickerSymbol));  
    }  
    return Long.valueOf(price);  
}
```
- 배포 서버의 redis 저장소를 확인해보니 종목 종가에 대한 데이터를 저장하지 않고 있었습니다.
![[Pasted image 20231204233609.png]]

즈