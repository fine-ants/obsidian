# TODO
- [ ] 반응형
	-  < 768px: Mobile
	- < 1024px: Tablet
	- >= 1024px: Desktop

- [ ] SSE 적용
	- [ ] Dashboard Page
	- [ ] Portfolio Page의 PortfolioHoldingsPieChart

- [ ] Optimization
	- [ ] Performance
	    - [x] FCP, LCP
	- [x] Accessibility
		- [ ] Navigate search result dropdown using arrow keys
		- [ ] Button (https://dequeuniversity.com/rules/axe/4.7/button-name)
		- [ ] Image (https://dequeuniversity.com/rules/axe/4.7/image-alt)
	- [ ] Layout Shift
		- [ ] SignInPage image layout shift
	- [ ] Bundle size 줄이기
	- [ ] Legacy JS Modern Browsers
        ![[legacy-js-modern-browsers.png]]
	- [ ] Text Compression
		- https://developer.chrome.com/docs/lighthouse/performance/uses-text-compression/?utm_source=lighthouse&utm_medium=devtools

- [ ] Test (E2E)

- [ ] Token Studios Plugin
	- [ ] https://docs.tokens.studio/getting-started
	- [ ] https://velog.io/@seo__namu/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EC%97%90-%EB%94%94%EC%9E%90%EC%9D%B8-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0#%EA%B7%B8%EB%9E%98%EC%84%9C-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C%EB%8A%94-%EB%AD%98-%ED%95%B4%EC%95%BC%ED%95%98%EB%8A%94%EB%8D%B0

## 목표
### Kakamotobi
- [x] OAuth 절차 변경
	- https://velog.io/@jkijki12/Spring-Boot-OAuth2-JWT-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EB%A6%AC%EA%B8%B0
- [ ] 검색 결과 무한 스크롤 적용
	 - From jay 
		- 무한 스크롤 api 구현되어 있음
		- 새 스크롤 api 방식에서 현재 데이터가 마지막인지(더 이상 스크롤해도 새 데이터가 안오는 상태) 아닌지를 data를 보고 확인 할 방법이 없음, 마지막에서 새로 스크롤하면 빈 데이터가 오는데 1회가 조금 아쉬움
			- 백엔드에서 추후에 추가도 가능하다 하셨음, 변경을 원하면 백엔드에 요청 가능
		
- [ ] IconButton refactor
	- [ ] 모바일 "내 포트폴리오" 페이지 포트폴리오 추가 버튼 수정 필요
		
### Jay
- 포트폴리오 페이지 SSE로 자주 랜더링되는 상태 최적화
	- 데스크탑은 완료.
	- 모바일 추가 필요
- https://patterns-dev-kr.github.io/design-patterns/compound-pattern/
	- 적용할 수 있는 부분 찾기
- build 될 때 api/proxy 파일들 빌드 안되게 하기
	- Header  로그인 상태를 서버에서 useConetxt로 보관중인 user를 사용하지 못해서 임시로 dynamic import 중인데 이부분 개선하기
		- useContext없이 가능 방향도 고려
- TICKER TAPE 추가하면 BasePage에 min-heightdp TICKER TAPE 높이 제거하기
- Header dynamic import 변경하기
	- 레이아웃 쉬프트생김
### Bakha

---
### TODO
- [ ] 서버 데이터와 MSW 데이터 및 핸들러 맞추기
- [ ] FCM
	- [ ] 첫 접속시 FCM SW가 설치되기 전에 에러 토스트가 발생해버리고 FCM Token 등록을 진행하지 않음 (새로고침 하기 전까지는).
	- [ ] FCM token 삭제 확인 (NotificationSettingsDialog)
	- [ ] UserContext 및 FCM Token 관련 리팩토링 (재렌더에 의한 불필요한 setupFCMToken 요청)
	- [ ] FCM token 오류 확인 필요
		- 종종 FCM으로부터 발급 받은 토큰이 `404 UNREGISTERED` 오류가 날 때 해당 토큰을 제거하고 새로운 토큰을 발급받아야 함.
		- https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode
	- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
- [ ] 로그인 페이지 "내 정보 기억하기" 구현
- [ ] Github PR bot 생각해보기
	- 릴리즈, 메인 배포시 PR 내용을 간단하게 요약해서 pr 만들어줄 수 있는 봇?
	- PR 내용으로 올라가는 이슈 번호 나열같은 기능
	
- [ ] 알림 설정 토글 브라우저별로(사파리 파이엎폭스 크롬) 테스트 해보기
- [ ] 포트폴리오 페이지 고정적인 값들 SSE를 통해서 계속 재랜더링중 최적화하면 좋을 듯
- [ ] Mutate 함수 실행 후 특정 로직이 이어서 작업되는 경우 Mutate가 실패 시 다음 동작을 안하도록 고려하기
- [ ] 파인앤츠 한글로 구글 검색 가능하도록
- [ ] 트레이딩뷰 한국 주식 제공 안하는 부분 해결 or 다른 방법 찾기
- [ ] zod 라이브러리 확인 하고 적용 가능 여부 판단하기

#### next
- favicon 라이트, 다크 적용
	- 다크만 적용되어있음

#### UI
- [ ] 모바일용 디자인 시스템 확인해서 공용 컴포넌트들 리팩터링
- [ ] Open Graph 디자인 요청하기

- [ ] skeleton UI,  error Fallback 반응형 작업한 것에 맞게 리팩터링
- [ ] TVTickerTapeWidget 유무에 따라서 BasePage 컴포넌트의 높이가 달라져야한다.
	- ActiveNotificationsPage의 경우 조금 이상하게 적용중(확인 필요)
- [ ] StockPage, 관심 종목 추가 페이지 아이템 선택 후 저장 버튼 일관성 있게 수정하게
	- 어느 곳은 저장(추가) 버튼이 있고 없는 어느 곳은 없음
#### 기타
- `Third-party cookie will be blocked. Learn more in the Issues tab.`
