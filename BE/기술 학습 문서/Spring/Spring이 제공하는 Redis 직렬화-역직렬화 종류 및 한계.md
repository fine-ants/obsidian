
## 1. Spring이 제공하는 Redis 직렬화-역직렬화의 종류와 한계
Spring이 제공하는 Redis 직렬화-역직렬화 종류는 다음과 같습니다.
- StringRedisSerializer
- JdkSerializationRedisSerializer
- GenericJackson2JsonRedisSerializer
- Jackson2JsonRedisSerializer

### StringRedisSerializer
- 문자열을 UTF-8로 인코딩해서 byte 배열로 변환함
- 간단한 문자열 처리를 위해서 사용되고 주로 key값 처리할때 사용됨

### JdkSerializationRedisSerializer
- 자바 기본 직렬화 기법인 JDK 직렬화를 사용함
- 별도 설정이 없으면 RedisTemplate, @Cacheable 등에서 기본적으로 사용됨
- JDK 직렬화 사용할 때 발생할 수 있는 문제를 그대로 경험하게 됨
	- 직렬화하는 객체의 클래스는 Serializable 인터페이스를 구현해야함
	- serialVersionUID 값을 가져야함
	- 클래스 정보가 

