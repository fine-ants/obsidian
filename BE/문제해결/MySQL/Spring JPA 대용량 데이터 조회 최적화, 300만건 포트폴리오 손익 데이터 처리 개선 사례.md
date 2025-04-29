
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
    order by p.createAt desc    
    """)  
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
성능 측정 결과를 보면 Pageable이 적용된 상태에서 약 3.44초가 소요되었습니다. 이전의 측정 할 수 없을 정도로 긴 시간을 대기하는 것에 비해서 나아진 결과입니다. 하지만 사용자 입장에서는 만족스러운 성능은 아닙니다.

현재 데이터베이스 쿼리 시간과 응답 시간이 비슷하기 때문에 쿼리하는 방식에 문제가 있다고 판단하였습니다. MySQL 클라이언트를 이용하여 다음 쿼리를 실행하여 우리가 구현한 쿼리의 실행 계획을 확인해봅니다.
```sql
explain select p.*, p2.* from portfolio_gain_history p  
    inner join portfolio p2 on p.portfolio_id = p2.id  
where p.portfolio_id = :portfolioId and p.create_at <= now()  
order by p.create_at desc  
limit 1;
```
![[Pasted image 20250428154555.png]]
portfolio 테이블(p2)
- 처리 방법: const 타입으로 조회
	- p2 테이블은 PRIMARY KEY(id)를 기준으로 단 하나의 레코드(rows)를 즉시 조회합니다.
- 특이사항: Using filesort
	- 일반적으로 const 조회에는 정렬이 필요 없지만, 전체 쿼리 결과를 합친 후 정렬이 추가로 발생한 것으로 보입니다.(JOIN 이후 정렬 때문)

portfolio_gain_history 테이블(p)
- 처리 방법: ref 타입으로 조회
	- portfolio_id 인덱스를 사용해서 주어진 portfolioId에 해당하는 레코드를 찾습니다.
- 읽은 예상 레코드 개수: 약, 1,453,931건
	- 상당히 많은 양의 데이터 중에서 조건(create_at <= now())을 설정하여 걸러야 합니다.
- 필터링 비율: 33.33%
	- 읽어온 데이터중 약 33.33%만 조건을 만족합니다.
- 특이사항: Using where
	- portfolio_id 인덱스만으로 모든 조건을 해결하지 못하고, 추가로 create_at <= now() 조건을 사용합니다.

위 실행계획을 요약하면 다음과 같습니다.
- portfolio 테이블은 PK 기반 단건 조회로 성능 문제가 없습니다.
- portfolio_gain_history 테이블은 portfolio_id 인덱스를 이용하지만, create_at 조건을 따로 평가해야 해서 추가적인 레코드 스캔이 필요합니다.
- 정렬(order by create_at desc)을 위해서 추가적인 파일 정렬(filesort)가 필요합니다.
- 쿼리가 3.5초가 소요되는 주요 원인은 create_at desc 정렬이 인덱스를 못타서 filesort가 발생했기 때문입니다.

성능 개선 방향
- portfolio_gain_history에 portfolio_id, create_at을 복합 인덱스로 구성하면, where 조건절과 order by 모두 인덱스만으로 처리할 수 있어서 filesort없이 최신 레코드 하나를 훨씬 빠르게 찾을 수 있습니다.

복합 인덱스 생성
portfolio_id, create_at 컬럼을 대상으로 인덱스를 생성합니다. 단, create_at 컬럼에 내림차순(desc)을 상세 설정합니다.
```java
ALTER TABLE portfolio_gain_history  
    ADD INDEX idx_portfolio_id_create_at (portfolio_id, create_at DESC);
```

복합 인덱스 생성 후 쿼리 실행
```java
explain select p.*, p2.* from portfolio_gain_history p  
    inner join portfolio p2 on p.portfolio_id = p2.id  
where p.portfolio_id = :portfolioId and p.create_at <= now()  
order by p.create_at desc  
limit 1;
```
![[Pasted image 20250428160229.png]]

portfolio 테이블(p2)
- 처리 방법: const 타입
	- portfolio 테이블은 id(PRIMARY KEY)를 기준으로 단건으로 즉시 조회합니다.
- 특이사항: 없음
	- 이전 결과와는 다르게 Using filesort가 없습니다.
	- JOIN 이후 정렬이나 추가적인 비용이 발생하지 않습니다.

portfolio_gain_history 테이블(p)
- 처리 방법: range 타입
	- 새로 생선한 복합 인덱스(idx_portfolio_id_create_at)을 사용하여 주어진 portfolio_id와 create_at 조건을 모두 인덱스 레벨에서 처리합니다.
	- range 스캔이란 범위 조건(<= now())을 만족하는 레코드를 인덱스 안에서 찾아가는 방식입니다.
- 예상되는 읽은 레코드 개수: 약 1,453,931건
	- 읽어야 할 데이터 개수는 많지만, 인덱스를 통해서 필요한 범위만 스캔합니다.
- 특이사항: Using index condition
	- 복합 인덱스를 이용해 일부 조건(portfolio_id)은 인덱스 레벨에서 평가하고, 나머지(create_at)은 인덱스에서 읽어온 값으로 필터링합니다. 
	- 레코드를 따로 읽지 않고 인덱스 컬럼만으로 where 절을 평가 시도합니다. (Index Condition PushDown, ICP)
	- 추가적인 데이터 레코드(row)를 읽는 비용이 감소됩니다.


> [!NOTE] Index Condition PushDown(ICP)
> MySQL이 테이블 레코드를 읽기 전에 인덱스에 이미 저장된 컬럼 값만을 이용해서 where 조건을 먼저 평가하는 최적화입니다. 필요없는 레코드는 아예 읽지 않고 건너 뛸 수 있게 합니다. 이로 인해서 디스크 I/O가 줄어들고 쿼리 성능이 좋아집니다
> 즉, 인덱스만으로 가능한 최대한 필터링하고 진짜 필요한 것만 디스크에서 레코드 데이터를 읽는 전략입니다.


복합 인덱스를 추가하면서 생긴 변화
- portfolio_id + create_at 조건절을 인덱스 레벨에서 모두 처리 가능하게 되었습니다.
- filesort가 제거되었고, Using Index Condition 덕분에 레코드를 디스크에서 추가로 읽지 않고 인덱스 레벨에서 처리할 수 있게 되었습니다.
- 복합 인덱스를 추가하면서 쿼리 성능이 향상되었습니다.


위와 같이 복합 인덱스를 설정한 상태에서 다시 API를 요청하여 성능 측적을 수행합니다.
![[Pasted image 20250428160917.png]]
실행 결과를 보면 기존 **3440ms에서 199ms로 17.2배 개선된 것**을 볼수 있습니다.

정리
- JPQL 쿼리 메서드에 Pageable 매개변수를 추가하여 1건의 데이터만 조회할 수 있도록 합니다.
- portfolio_id, create_at 컬럼을 대상으로 복합 인덱스를 구성하여 쿼리 시간을 단축시킵니다.
- 성능 측정 결과 기존 3440ms에서 199ms로 약 17.2배 개선되엇습니다.

## References
- https://jojoldu.tistory.com/474