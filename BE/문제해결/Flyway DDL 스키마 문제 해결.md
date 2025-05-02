
## 배경
쿼리 성능 개선을 위하여 다음과 같은 복합 인덱스를 생성하는 파일을 추가하였습니다.
V4__add_index_portfolio.sql
```sql
CREATE INDEX idx_member_create_at ON portfolio (member_id, create_at DESC);
```

위와 같이 파일을 추가한 다음에 spring 서버를 실행하였습니다.
![[Pasted image 20250502122612.png]]
실행 결과 4번째 버전인 add index portfolio에서 마이그레이션이 실패했다고 합니다. 데이터베이스에는 spring 서버가 실행전에 이미 복합 인덱스가 생성된 상태였습니다.

저는 데이터베이스에 이미 복합 인덱스가 생성되었다고 판단하여 복합 인덱스를 제거하였습니다.
```
drop index idx_member_create_at on portfolio;
```

그리고 다시 spirng 서버를 실행하여 flyway를 실행시켜봅니다.
![[Pasted image 20250502130506.png]]
실행 결과를 보면 여전히 마이그레이션 4번째 버전이 실패한 것을 볼수 있습니다.

## 원인
Flyway로 생성된 flyway_schema_history 테이블에는 스키마 변경 내역이 저장되는데 만약 V4 스크립트 파일을 마이그레이션하다가 오류가 발생하게 된다면, flyway_schmea_history 테이블에는 V4에 대한 실패 내역이 저장됩니다. 실제로 저같은 경우 데이터베이스에 이미 idx_member_create_at 복합 인덱스를 생성해둔 상태에서 V4 스크립트를 실행했지만 인덱스 생성이 중복되어 실패 내역이 flyway_schema_history 테이블에 저장됩니다. 그리고 본인이 데이터베이스에 이미 존재하는 복합 인덱스를 제거하고 다시 spring 서버를 실행해도 이미 flyway_schema_history 테이블에는 V4 스크립트에 대한 실패 내역이 남아있기 때문에 V4 버전은 flyway가 실행하지 않습니다.

다음은 이미 V4 버전에 대한 실패 내역이 저장된 flyway_schema_history 테이블입니다.
![[Pasted image 20250502131131.png]]
위 실행 결과를 보면 V4 버전에 대해서 success=0으로 저장되어 있는 것을 볼수 있습니다. 이미 실패 내역이 존재하기 때문에 아무리 실패의 원인이 되는 복합 인덱스를 제거해도 여전히 flyway가 실행되지 않는 것입니다.


## 해결 방법
위 문제를 해결할 첫번째 방법은 DB에 직접 접근하여 flyway_schema_history 테이블에 실패한 내역이 저장된 행을 직접 삭제합니다. 하지만 이 방법 같은 경우에는 별도로 DB에 접근해서 데이터를 삭제하는 번거로운 과정을 거쳐야 합니다. 이는 flyway 도구를 도입한 취지와 맞지 않을 수 있습니다.

두번째 방법은 spring에서 자동으로 repair 기능이 실행되도록 할 수 있습니다. **flyway repair 기능은 Flyway의 마이그레이션 메타데이터 테이블(flyway_schema_history)을 복구하는 명령입니다.** 
- 실패한 마이그레이션 레코드 제거
- 잘못된 체크섬(파일이 수정됨)을 재계산
- flyway_schema_history 테이블의 이상 상태를 정리

Spring의 FlywayMigrationStrategy
Spring에서는 FlywayMigrationStrategy(org.springframework.boot.autoconfigure.flyway)를 제공하고 있습니다. 해당 인터페이스는 Flyway 마이그레이션을 초기화하는데 사용됩니다. FlywayMigrationStrategy 타입을 스프링 빈으로 등록하여 마이그레이션 동작을 재정의 할 수 있습니다.

마이그레이션 동작을 재정의해서 flyway의 마이그레이션 작업이 실행되기 직전에 직접 repair 기능을 수행한다면 flyway에 문제가 발생해도 적절한 상태에서 마이그레이션할 수 있습니다. 설정 코드는 다음과 같습니다.
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

## 실행 결과
데이터베이스에 복합 인덱스가 제거된 상태에서 다시 spring 서버를 실행해보겠습니다. 서버를 실행한 다음에 flyway_schema_history 테이블의 데이터를 확인해보겠습니다.
![[Pasted image 20250502134029.png]]
실행 결과를 보면 V4 버전에 대해서 기존 success 컬럼의 값이 0에서 1로 변경된 것을 볼수 있습니다.

portfolio 테이블에 인덱스가 정상적으로 생성되었는지 확인해보겠습니다.
```sql
show index from portfolio;
```
![[Pasted image 20250502134140.png]]
위 실행 결과를 보면 portfolio에 정상적으로 복합 인덱스 idx_member_create_at이 생성된 것을 볼수 있습니다.

## References
- https://backend-repository.tistory.com/159