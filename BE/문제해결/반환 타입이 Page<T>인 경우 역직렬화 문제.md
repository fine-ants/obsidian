
## 배경
다음과 같은 서비스 메서드가 존재합니다. 해당 서비스는 포트폴리오 이름 목록을 페이징하여 조회하는 서비스입니다. 
```java
@Transactional(readOnly = true)  
@Cacheable(value = "myAllPortfolioNames", key = "#memberId + '_' + #pageable.pageNumber + '_' + #pageable.pageSize")  
@Secured("ROLE_USER")  
public Page<PortfolioNameItem> getPagedPortfolioNames(@NotNull Long memberId, @NotNull Pageable pageable) {  
    Page<Portfolio> page = portfolioRepository.findAllByMemberIdAndPageable(memberId, pageable);  
    List<PortfolioNameItem> items = page.stream()  
       .map(PortfolioNameItem::from)  
       .toList();  
    return new PageImpl<>(items, pageable, page.getTotalElements());  
}
```

위와 같은 서비스 메서드를 통하여 redis에 결과를 직렬화하여 캐싱한 다음에 추후 다시 접근하여 캐싱된 데이터를 다시 역직렬화시 다음과 같은 에러가 발생합니다.
```
org.springframework.data.redis.serializer.SerializationException: Could not read JSON:Cannot construct instance of `org.springframework.data.domain.PageImpl` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (byte[])"["org.springframework.data.domain.PageImpl",{"content":["java.util.Collections$UnmodifiableRandomAccessList",[["co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem",{"id":["java.lang.Long",11],"name":"portfolio10","dateCreated":["java.time.LocalDateTime","2025-05-07T12:08:34.191867"]}],["co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem",{"id":["java.lang.Long",10],"name":"portfolio9","dateCreated":["java.time.LocalDateTime","2025-05-07T12:08:34.184534"]}]"[truncated 1993 bytes]; line: 1, column: 46]
```

redis에 저장된 json 데이터는 다음과 같습니다.
```
127.0.0.1:6379> get myAllPortfolioNames::1_0_10
"[\"org.springframework.data.domain.PageImpl\",{\"content\":[\"java.util.Collections$UnmodifiableRandomAccessList\",[[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",11],\"name\":\"portfolio10\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.081465\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",10],\"name\":\"portfolio9\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.076012\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",9],\"name\":\"portfolio8\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.069851\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",8],\"name\":\"portfolio7\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.061471\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",7],\"name\":\"portfolio6\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.052325\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",6],\"name\":\"portfolio5\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.043378\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",5],\"name\":\"portfolio4\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.031844\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",4],\"name\":\"portfolio3\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.022183\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",3],\"name\":\"portfolio2\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:46.001476\"]}],[\"co.fineants.api.domain.portfolio.domain.dto.response.PortfolioNameItem\",{\"id\":[\"java.lang.Long\",2],\"name\":\"portfolio1\",\"dateCreated\":[\"java.time.LocalDateTime\",\"2025-05-07T12:18:45.990496\"]}]]],\"pageable\":[\"org.springframework.data.domain.PageRequest\",{\"pageNumber\":0,\"pageSize\":10,\"sort\":[\"org.springframework.data.domain.Sort\",{\"empty\":false,\"sorted\":true,\"unsorted\":false}],\"offset\":0,\"paged\":true,\"unpaged\":false}],\"last\":false,\"totalElements\":11,\"totalPages\":2,\"first\":true,\"size\":10,\"number\":0,\"sort\":[\"org.springframework.data.domain.Sort\",{\"empty\":false,\"sorted\":true,\"unsorted\":false}],\"numberOfElements\":10,\"empty\":false}]"
```

## 원인
에러가 발생한 원인은 페이징 라이브러리인 PageImpl 클래스가 기본 생성자가 없기 때문에 Jackson이 JSON을 자바 객체로 역직렬화하지 못합니다.

## 해결 방법
