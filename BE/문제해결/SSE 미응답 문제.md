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

즉, SSE 응답을 하지 못한 원인은 포트폴리오 상세 조회 데이터를 만들려고 하다가 redis에 종가 데이터를 저장하지 않아서 실행 중 IllegalArgumentException이 발생한 것이었고 이 예외가 발생했음에도 스케줄링 안에 쓰레드에서 실행되어 예외가 로깅되지 않았던 것이었습니다.

발견한 계기는 쓰레드풀에 넣지 않고 실행했을때 GlobalExceptionHandler로 로깅할 수 있었습니다.

```
@GetMapping  
public SseEmitter readMyPortfolioStocks(@PathVariable Long portfolioId) {  
    SseEmitter emitter = new SseEmitter(1000L * 30);  
    emitter.onTimeout(emitter::complete);  
  
    try {  
       emitter.send(SseEmitter.event()  
          .data(portfolioStockService.readMyPortfolioStocks(portfolioId, lastDayClosingPriceManager))  
          .name("sse event - myPortfolioStocks"));  
       emitter.complete();  
    } catch (IOException e) {  
       log.error(e.getMessage(), e);  
    }  
  
    // 장시간 동안에는 스케줄러를 이용하여 지속적 응답  
    // if (stockMarketChecker.isMarketOpen(LocalDateTime.now())) {  
    //     scheduleSseEventTask(portfolioId, emitter, false);    // } else {    //     scheduleSseEventTask(portfolioId, emitter, true);    // }    return emitter;  
}
```
![[Pasted image 20231204234122.png]]

#### scheduleAtFixedRate vs. scheduleWithFixedDelay
scheduleAtFixedRate(command, initialDelay, period, unit)
1. initialDelay 후에 period 간격으로 command 실행
2. 어떤 이유(예: 가비지 컬렉션 또는 기타 백그라운드 활동)로 인해 실행이 지연되면 두번 이상 실행될 수 있습니다.
	- ex) scheduleAtFixedRate(command, 1, 2, TimeUnit.SECONDS) 일 때 실행시간이 12시 00분 00초이면 1초 후인 12시 00분 01초에 command가 실행되고 다음은 12시 00분 03초, 5초, 7초입니다. command의 종료 여부와 관계가 없습니다.

scheduleWithFixedDelay(command, initialDelay, delay, unit)
1. initialDelay 후에 delay 간격으로 command 실행
2. command가 종료된 후를 기준으로 delay 간격으로 command 실행
	- cheduleWithFixedDelay(command, 1, 2, TimeUnit.SECONDS) 일 때 실행시간이 12시 00분 00초라면 1초 후인 12시 00분 01초에 command가 실행되고 그후 command가 10초 후에 종료되었다면 다음은 12시 00분 13초입니다. 그후 command가 2초후 종료되었다면 다음은 12시 00분 17초입니다.


### 해결방법
- 5초에 한번씩 현재가 및 종가 가격을 갱신하는 부분에서 문제를 해결합니다.
- 종가 갱신하는 부분은 쓰레드 풀을 사용하여 테스크를 실행하는데 이 부분의 테스크들을 한번에 요청하여 open api에서 초과 요청으로 인해서 가격 문의가 거부당하였습니다. 따라서 쓰레드풀의 크기를 1로 설정하여 각각의 테스크가 0.2초 간격으로 한개씩 전송하도록 합니다.
```
public class KisService {  
    private static final String SUBSCRIBE_PORTFOLIO_HOLDING_FORMAT = "/sub/portfolio/%d";  
    private static final ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);
```
- KisService 클래스의 executorService 쓰레드풀의  크기를 1로 설정하여 각각의 테스크가 0.2초 간격으로 수행하도록 합니다.
