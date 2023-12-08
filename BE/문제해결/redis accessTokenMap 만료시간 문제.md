
## 상황
redis에 저장되는 데이터중 kis open api 서버의 accessToken을 하루동안 보관하고 있습니다. 그러나 다음 사진과 같이 access_token_token_expired의 값이 만료되었음에도 불구하고 데이터가 살아있는 것을 확인하였습니다.
 
<img width="857" alt="image" src="https://github.com/fine-ants/FineAnts-was/assets/33227831/36682280-688f-40be-91f7-165c1ca57d06">

위와 같은 이유로 로컬 환경에서 서버 실행시 만료된 accessTokenMap이 살아있어서 현재가 및 종가를 갱신하지 못하는 상황입니다.

redis 터미널에서 TTL 명령어로 수명을 확인했을때 7290초의 수명이 남은 것을 확인할 수 있었습니다.
<img width="344" alt="image" src="https://github.com/fine-ants/FineAnts-was/assets/33227831/df4cbafe-5cdb-42ce-b540-5d6819f1ff31">


## 원인
kis open api 서버로부터 발급받은 액세스 토큰의 만료시간은 발급받은 시점으로부터 22시간동안 유지되지만 만료초(expires_in)은 24시간(86400)이다. 그런데 KisRedisService 객체는 accessTokenMap을 저장시 expires_in 데이터를 가지고 저장하기 때문에 토큰이 만료되었음에도 불구하고 2시간동안 살아있기 때문에 갱신이 안되었던 것이다.
 ```
public void setAccessTokenMap(Map<String, Object> accessTokenMap) {
		long expiresIn = ((Integer)accessTokenMap.get("expires_in")).longValue();
		String json;
		try {
			json = objectMapper.writeValueAsString(accessTokenMap);
		} catch (JsonProcessingException e) {
			throw new RuntimeException(e);
		}
		redisTemplate.opsForValue().set(ACCESS_TOKEN_MAP_KEY, json, Duration.ofSeconds(expiresIn));
	}
```

## 해결방법
- accessTokenMap을 redis에 access_token_token_expired 프로퍼티의 날짜까지만 저장하도록 변경합니다.
```
public void setAccessTokenMap(Map<String, Object> accessTokenMap, LocalDateTime now) {  
    String json;  
    try {  
       json = objectMapper.writeValueAsString(accessTokenMap);  
    } catch (JsonProcessingException e) {  
       throw new RuntimeException(e);  
    }  
    String access_token_token_expired = (String)accessTokenMap.get("access_token_token_expired");  
    long exp = Duration.between(now,  
          LocalDateTime.parse(access_token_token_expired, formatter))  
       .toSeconds();  
    redisTemplate.opsForValue().set(ACCESS_TOKEN_MAP_KEY, json, Duration.ofSeconds(exp));  
}
```
