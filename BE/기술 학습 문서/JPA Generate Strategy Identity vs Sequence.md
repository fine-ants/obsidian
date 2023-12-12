### sequence 사용
새로운 행을 삽입하기 전에 데이터베이스에게 다음 sequence 값을 요청합니다. 그리고 나서 행을 반환된 sequence 값(ID로써 사용)과 함께 삽입합니다.

### Identity 사용
ID 값을 지정하지 않고 행을 삽입합니다. 행을 삽입하고 나서 데이터베이스에게 마지막으로 생성된 ID에 대하여 요청합니다.

두 경우 모두 쿼리의 수는 동일합니다. 그러나 **Hibernate는 기본적인 전략으로 sequence 생성 전략을 더 효율적인 전략으로 사용**합니다. 실제로 다음 sequence 값을 요청할 때, 메모리에 50(기본값, 설정 가능)개의 다음 값을 유지하고, 다음 50개의 삽입에 이 50개의 다음값을 사용합니다. 50개의 삽입후에만 데이터베이스로 이동하여 50개의 다음값을 가져옵니다. 이렇게 하면 자동 ID 생성에 필요한 SQL 쿼리수가 엄청나게 줄어듭니다.

Identity 전략은 이러한 최적화를 허용하지 않고 있습니다.

즉, 정리하면 두 쿼




### References
- [Hibernate IDENTITY vs SEQUENCE entity identifier generators](https://stackoverflow.com/questions/17780394/hibernate-identity-vs-sequence-entity-identifier-generators) 