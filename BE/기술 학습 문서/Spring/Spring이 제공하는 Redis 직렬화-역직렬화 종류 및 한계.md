

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
	- 직렬화된 값에 클래스 정보가 포함되서 용량을 많이 차지하게 됨
	- 클래스가 패키지 이동이나 클래스 이름이 변경되면 역직렬화에 실패할 수 있음

### GenericJackson2JsonRedisSerializer
- ObjectMapper를 사용하여 직렬화를 시도함
- 설정에 따라서 직렬화 할때 클래스 정보가 포함되어 여러 문제가 생길 수 있음
- Json 객체를 그대로 바이트 배열로 변환하여 저장하기 때문에 저장공간이 많이 사용됨

### Jackson2JsonRedisSerializer
- 객체를 JSON 형태로 직렬화하고 클래스 정보가 포함되지 않지만 Serializer 객체가 항상 타입을 지정해야함
- 모든 캐시 대상 객체별로 Jackson2JsonRedisSerializer 객체를 생성해야 함
- JSON 객체를 그대로 바이트 배열로 변환하여 저장하므로, 저장 공간이 많이 사용됨
- 의도치 않은 ClassCastException 문제가 발생할 수 있음. 왜냐하면 클래스 정보가 포함되어 있지 않기 때문이다.



## References
- https://mangkyu.tistory.com/402
