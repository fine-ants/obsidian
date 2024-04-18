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
- [ ] 최근 로그인한 방식 UI 제공
### Jay

### Bakha
- [ ] Watchlist breadcrumb 이름 수정
- [ ] Portfolio List Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [ ] Watchlist Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [ ] FineAnts logo 이미지 사이즈 조정 필요 (40x40인 원안에 들어가도 자연스럽게)
	- [ ] `/src/assets/icons/logo/ic_fineants.svg` 해당 경로 svg 사이즈 조절이 필요해 보임
#### 포트폴리오 
- 추가 Dialog
	- [ ] 제목 폰트 적용 안됨
	- [ ] 예산 및 금액 input에 `,` 적용
	- [ ] 예산, 목표 수익률, 최대 손실율 입력한 뒤 예산을 변경하면,
		- 목표 수익률(%)이 변하는데 %가 아니라 금액이 변하도록 수정
		- 최대 손실율 금액도 변하도록 수정
	- 예산 입력했지만 목표 수익률 및 최대 손실율을 입력안해도 되게 수정
- [ ] 포트폴리오 삭제 후 portfolioList query invalidate 필요
- [ ] 포트폴리오 삭제 Dialog
	- "항목을" -> "포트폴리오를"로 변경
#### Watchlist
- [ ] Watchlist Table 디자인 수정
	- [x] 별 제거
	- *Watchlist Table 단일 삭제 API 불필요*
- Watchlist 변동률 정렬이 적용이 안됨
	- 서버에서 rate가 0으로 들어와서 안되는 것이었음 change로 바꾸면 값이 있어서 정렬됨
- [ ] 와치리스트 리스트, 와치리스트 페이지 min-height 제거


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
- **"안되는" 포트폴리오(ErrorBoundary component가 적용 됨)에서 "되는" 포트폴리오로 이동할 때 holdings 패널이 그대로 ErrorBoundary component가 남아있음. "새로고침" 버튼을 눌러야 갱신 됨.**
- [ ] "PortfolioAddOrEditDialog" 목표 수익률 및 최대 손실율 input 및 계산 문제.
- [ ] 알림 설정 토글 브라우저별로(사파리 파이엎폭스 크롬) 테스트 해보기
- [ ] 포트폴리오 페이지 고정적인 값들 SSE를 통해서 계속 재랜더링중 최적화하면 좋을 듯
#### UI
- [ ] Table header column title (Window/MacOS Font 확인 필요)
	- Table header column의 가로 사이즈 문제인듯?(반응형 하며 수정하기)
- [ ] 포트폴리오 페이지에 종목 테이블에 `항목추가` 버튼 눌러서 종목 추가 새 input이 생기면서 레이아웃 쉬프트 생김
- [ ] 3
#### 기타
- `Third-party cookie will be blocked. Learn more in the Issues tab.`

