# TODO
- [x] **알림 기능**

- [ ] Test (E2E)

- [ ] 반응형

- [ ] SSE 적용
	- [ ] Dashboard Page
	- [ ] Portfolio Page의 PortfolioHoldingsPieChart

- [ ] Optimization
	- [ ] Performance
	    - [ ] FCP, LCP
	- [ ] Accessibility
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

## 목표
### Kakamotobi
- [ ] README update (알림 관련 내용 추가)
- [ ] PortfolioNotificationListTable에서 알림 활성화/비활성화할 때 정렬 순서 바뀌는 문제
	- `lastUpdated` --> `createdAt`으로 변경하면 될 듯함 (BE 요청)
### Jay
- [ ] 관심 종목 페이지 스켈레톤 사이즈 변경
	- 실제 테이블 보다 넓은 범위를 가지고 있어서 로딩이 끝나고 실제 테이블이 보였을 때 부자연스럽다
- [ ] PieChartLegendSkeleton 구현
### Bakha
- [ ] Watchlist breadcrumb 이름 수정
- [ ] Portfolio List Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [ ] Watchlist Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [ ] FineAnts logo 이미지 사이즈 조정 필요 (40x40인 원안에 들어가도 자연스럽게)
	- [ ] `/src/assets/icons/logo/ic_fineants.svg` 해당 경로 svg 사이즈 조절이 필요해 보임
#### Watchlist
- [ ] Watchlist Table 디자인 수정
	- [x] 별 제거
	- [ ] Watchlist Table 단일 삭제 API 불필요 (BE 전달)

---
### TODO
- [ ] FCM
	- [ ] 첫 접속시 FCM SW가 설치되기 전에 에러 토스트가 발생해버리고 FCM Token 등록을 진행하지 않음 (새로고침 하기 전까지는).
	- [ ] FCM token 삭제 확인 (NotificationSettingsDialog)
	- [ ] UserContext 및 FCM Token 관련 리팩토링 (재렌더에 의한 불필요한 setupFCMToken 요청)
	- [ ] FCM token 오류 확인 필요
		- 종종 FCM으로부터 발급 받은 토큰이 `404 UNREGISTERED` 오류가 날 때 해당 토큰을 제거하고 새로운 토큰을 발급받아야 함.
		- https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode
	- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
- [ ] 로그인 페이지 "내 정보 기억하기" 구현
- [ ] GIthub PR bot 생각해보기
	- 릴리즈, 메인 배포시 PR 내용을 간단하게 요약해서 pr 만들어줄 수 있는 봇?
	- PR 내용으로 올라가는 이슈 번호 나열같은 기능
- [ ] 알림 설정 토글 브라우저별로(사파리 파이엎폭스 크롬) 테스트 해보기
- [ ] 포트폴리오 페이지 고정적인 값들 SSE를 통해서 계속 재랜더링중 최적화하면 좋을 듯
- [ ] Synchronize MSW Handlers
#### UI
- [ ] Table header column title (Window/MacOS Font 확인 필요)
	- Table header column의 가로 사이즈 문제인듯?(반응형 하며 수정하기)
- [ ] 포트폴리오 페이지에 종목 테이블에 `항목추가` 버튼 눌러서 종목 추가 새 input이 생기면서 레이아웃 쉬프트 생김
- [ ] PortfolioAddOrEditDialog
	- [ ] 목표 수익률 및 최대 손실율 toast를 input 아래 문구로 변경
	- [ ] Submit button `disabled` 조건 추가 (목표 수익률 및 최대 손실율)

#### 기타
- `Third-party cookie will be blocked. Learn more in the Issues tab.`
