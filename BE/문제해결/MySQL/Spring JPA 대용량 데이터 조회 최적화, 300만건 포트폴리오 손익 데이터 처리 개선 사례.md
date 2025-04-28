
## 배경
- 데이터베이스에 포트폴리오 손익 내역(PortfolioGainHistory) 데이터가 300만개인 상태에서 포트폴리오 종목 조회시 응답 시간이 길어지는 문제가 발생하였습니다.
- api : /api/portfolio/:portfolioId/holdings

포트폴리오 손익 내역 데이터를 가져오기 위한 JPQL은 다음과 같았습니다.
```java
@Query(value = """  
    select p, p2 from PortfolioGainHistory p    
	    inner join Portfolio p2 on p.portfolio.id = p2.id  
    where p.portfolio.id = :portfolioId and p.createAt <= :createAt    
    order by p.createAt desc    
    """)  
List<PortfolioGainHistory> findFirstLatestPortfolioGainHistory(
	@Param("portfolioId") Long portfolioId, 
	@Param("createAt") LocalDateTime createAt);
```

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
300만건의 데이터를 한번에 가져와 메모리에 로드하는 문제를 해결하기 위해서 다음과 같이 수행합니다.
- @Query와 Pageable을 활용하여 첫번째 데이터만 가져옵니다. Pageable을 활용하면 쿼리에서 첫번째 페이지만 가져오도록 할 수 있습니다.

변경된 코드는 다음과 같습니다.
```java
@Query(value = """  
    select p, p2 from PortfolioGainHistory p    
    inner join Portfolio p2 on p.portfolio.id = p2.id  
    where p.portfolio.id = :portfolioId and p.createAt <= :createAt    
    order by p.createAt desc    """)  
List<PortfolioGainHistory> findFirstLatestPortfolioGainHistory(  
    @Param("portfolioId") Long portfolioId, 
    @Param("createAt") LocalDateTime createAt, 
    Pageable pageable);
```

위 코드를 이용하는 클라이언트 코드는 다음과 같이 변경됩니다.
```java
PortfolioGainHistory history =  
    portfolioGainHistoryRepository.findFirstLatestPortfolioGainHistory(  
          portfolio.getId(), LocalDateTime.now(), PageRequest.of(0, 1))  
       .stream()  
       .findAny()  
       .orElseGet(() -> PortfolioGainHistory.empty(portfolio));
```


포트폴리오 종목 조회 API(/api/portfolio/:portfolioId/holdings)를 요청하여 300만건의 데이터가 존재하는 상태에서 성능 측정을 수행해봅니다.
![[Pasted image 20250428145313.png]]
성능 측정 결과를 보면 Pageable이 적용된 상태에서 약 3.44초가 소요되었습니다.

MySQL 클라이언트를 이용하여 다음 쿼리를 실행하여 우리가 구현한 쿼리의 실행 계획을 확인해봅니다.
```sql
explain select p.*, p2.* from portfolio_gain_history p  
    inner join portfolio p2 on p.portfolio_id = p2.id  
where p.portfolio_id = :portfolioId and p.create_at <= now()  
order by p.create_at desc  
limit 1;
```
![[Pasted image 20250428154555.png]]
- p2는 ID기반 조회로 빠릅니다.
- p는 portfolio_id 인덱스를 이용해서 145만건을 탐색합니다.
- where p.create_at <= now() 필터링 후, order by p.create_at desc 정렬이 필요해서 전체 데이터에 대해 filesort를 발생시켰습니다.
- limit 1이 있지만, filesort 후에야 첫번째를 정할 수 있으므로, 여전히 전체 레코드를 정렬해야합니다.
- **즉, 3.5가 소요되는 주요 원인은 create_at desc 정렬이 인덱스를 못타서 filesort 발생했기 때문입니다.**

다음과 같이 복합 인덱스를 생성합니다.
```java
ALTER TABLE portfolio_gain_history  
    ADD INDEX idx_portfolio_id_create_at (portfolio_id, create_at DESC);
```

복합 인덱스를 생성한 상태에서 다시 쿼리를 실행합니다.
```java
explain select p.*, p2.* from portfolio_gain_history p  
    inner join portfolio p2 on p.portfolio_id = p2.id  
where p.portfolio_id = :portfolioId and p.create_at <= now()  
order by p.create_at desc  
limit 1;
```
![[Pasted image 20250428160229.png]]
실행 결과를 보면 where 절의 범위 조건을 다룰때 idx_portfolio_id_create_at 인덱스를 사용하는 것을 볼수 있습니다.

위와 같이 복합 인덱스를 설정한 상태에서 다시 API를 요청하여 성능 측적을 수행합니다.
![[Pasted image 20250428160917.png]]
실행 결과를 보면 기존 **3.44초에서 199ms로 17.2배 개선된 것**을 볼수 있습니다.

정리
- JPQL 쿼리 메서드에 Pageable 매개변수를 추가하여 1건의 데이터만 조회할 수 있도록 합니다.
- portfolio_id, create_at 컬럼을 대상으로 복합 인덱스를 구성하여 쿼리 시간을 단축시킵니다.