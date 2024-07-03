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
- [ ] Demolition `useImageInput` `initialImageUrl` `null` 도 받을 수 있도록 수정
- [x] OAuth 절차 변경
	- https://velog.io/@jkijki12/Spring-Boot-OAuth2-JWT-%EC%A0%81%EC%9A%A9%ED%95%B4%EB%B3%B4%EB%A6%AC%EA%B8%B0
- [ ] 포트폴리오 상세 테이블 변동률 디자인은 둘다 %인데 개발은 금액 + %임 확인 필요
- [ ] 종목 페이지 반응형
### Jay
- 포트폴리오 텝 폰트사이즈
- CardTable 페이지네이션 최소 높이(화면의 최하단으로)
- 포트폴리오 상세 holdings 반응형 작업
	- add/edit/read
- toolbar에 페이지 개수 수정 select 가로 크기 수정
### Bakha
- [ ] FineAnts logo 이미지 사이즈 조정 필요 (40x40인 원안에 들어가도 자연스럽게)
	- [ ] `/src/assets/icons/logo/ic_fineants.svg` 해당 경로 svg 사이즈 조절이 필요해 보임
- [ ] StockPage 관심 종목 설정 Dialog
	- [ ] Debounce 적용
	- [ ] To BE: 같은 watchlist가 여러개 발생함
#### Watchlist
- [ ] Watchlist Table 디자인 수정
	- [ ] Watchlist Table 단일 삭제 API 불필요 (BE 전달)

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
- [ ] 모바일 "내 포트폴리오" 페이지 포트폴리오 추가 버튼 수정 필요
- [ ] 모바일 "내 포트폴리오" 페이지  정렬 drawer 뒤로가기 디자인 추가될 예정
- [ ] drawer 중복적으로 사용하는 styled component 고민해 보기
- [ ] form으로 감싸진 곳에서 submit을 하면 화면 새로고침이 일어난다. SPA스럽지 못한 기분?
	- ex) 전체 관심 목록 페이지 이름 변경	
- [ ] 포트폴리오 CardTable 종목명 옆에 tickerSymbol 빠짐
- [ ] Button 컴포넌트 gap 디자인과 구현 상태 확인이 필요
- [ ] PortfolioListPage에서 isEmpty 없이 구현 가능 할 것 같음
- [ ] Mutate 함수 실행 후 특정 로직이 이어서 작업되는 경우 Mutate가 실패 시 다음 동작을 안하도록 고려하기
#### UI
- [ ] Design System에 있는 IconButton과 실제 디자인에 적용된 IconButton이 동일하지 않음 (to 디자인).
- [ ] Table header column title (Window/MacOS Font 확인 필요)
	- Table header column의 가로 사이즈 문제인듯?(반응형 하며 수정하기)
- [ ] 포트폴리오 페이지에 종목 테이블에 `항목추가` 버튼 눌러서 종목 추가 새 input이 생기면서 레이아웃 쉬프트 생김
- [ ] 모바일용 디자인 시스템 확인해서 공용 컴포넌트들 리팩터링
- [ ] 포트폴리오 리스트 페이지
	- 정렬 기능 구현 필요 (디자인 추가 되면)
	- carTableToolbar에 삭제 버튼 사이즈 수정 필요 (디자인 확인 필요)
	- 포트폴리오 리스트 페이지 header에 추가 
- [ ] Open Graph 디자인 요청하기

- [ ] skeleton UI,  error Fallback 반응형 작업한 것에 맞게 리팩터링

#### 기타
- `Third-party cookie will be blocked. Learn more in the Issues tab.`
