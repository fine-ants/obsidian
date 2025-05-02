
## 배경
쿼리 성능 개선을 위하여 다음과 같은 복합 인덱스를 생성하는 파일을 추가하였습니다.
V4__add_index_portfolio.sql
```sql
CREATE INDEX idx_member_create_at ON portfolio (member_id, create_at DESC);
```

위와 같이 파일을 추가한 다음에 spring 서버를 실행하였습니다.
![[Pasted image 20250502122612.png]]
실행 결과 4번째 버전인 add index portfolio에서 마이그레이션이 실패했다고 합니다. 데이터베이스에는 spring 서버가 실행전에 이미 복합 인덱스가 생성된 상태였습니다.

## 원인


## 해결 방법

```java
@Configuration  
public class FlywayConfig {  
  
    @Bean  
    public FlywayMigrationStrategy flywayMigrationStrategy() {  
       return flyway -> {  
          flyway.repair();  
          flyway.migrate();  
       };  
    }  
}
```


## References
- https://backend-repository.tistory.com/159