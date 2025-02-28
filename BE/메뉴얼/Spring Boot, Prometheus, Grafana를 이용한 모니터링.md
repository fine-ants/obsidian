
## 매트릭 설정
### 의존성 설정
```gradle
implementation 'org.springframework.boot:spring-boot-starter-web' 
implementation 'org.springframework.boot:spring-boot-starter-actuator' 
implementation 'io.micrometer:micrometer-registry-prometheus'
```

### actuator API 목록 확인
위와 같이 의존성 설정 후 서버를 실행합니다. 그리고 "http://localhost:8080/actuator" 경로로 요청합니다. 실행 결과는 다음과 같습니다. 실행 결과를 보면 현재 actuator가 제공하는 엔드포인트 목록을 확인할 수 있습니다. 클라이언트는 필요한 메트릭을 제공하는 API에 접속해서 메트릭 데이터를 확인할 수 있습니다.
```json
{
	"_links": {
		"self": {
			"href": "http://localhost:8080/actuator",
			"templated": false
		},
		"health": {
			"href": "http://localhost:8080/actuator/health",
			"templated": false
		},
		"health-path": {
			"href": "http://localhost:8080/actuator/health/{*path}",
			"templated": true
		}
	}
}
```

### 애플리케이션 프로퍼티 설정
