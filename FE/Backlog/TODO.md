# TODO

- [ ] **알림 기능**

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
- [ ] Demolition
	- [ ] formatTickValue
- [ ] FineAnts
	- [ ] FCM service worker 문제 해결
	- [ ] 최신 @fineants/demolition package 적용
	- [ ] README update (알림 관련 내용 추가)
### Jay
- [ ] Icon
	- [ ] Hover 필요한 icon 사용처 수정 (`hoverColor` prop 사용)
- [ ] TextButton, IconButton 사용처에 맞게 추가하기
- [ ] DashboardPage 포트폴리오 비중 차트 % 소수점 안보임.
- [ ] 매입 이력 추가했는데 현금이 부족 하면 400 Bad Request (사용자 입장에서는 아무 일이 안일어남). 토스트 피드백 제공.
- [ ] **포트폴리오 상세 페이지**
	- [ ] GET 요청에 대한 invalidate이 안되는 듯함.
	- [ ] 특정 포트폴리오 페이지에서 다른 포트폴리오 페이지로 이동 했을 때 "평가 금액"과 총 손익" 데이터가 이전 포트폴리오 "평가 금액" 및 "총 손익"이 남아있다 (SSE 연결 안됐을 때).

### Bakha
- [x] Watchlist breadcrumb 이름 수정
- [x] Portfolio List Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [x] Watchlist Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [x] FineAnts logo 이미지 사이즈 조정 필요 (40x40인 원안에 들어가도 자연스럽게)
#### 포트폴리오 
- 추가 Dialog
	- [x] 제목 폰트 적용 안됨
	- [ ] 예산 및 금액 input에 `,` 적용
	- [ ] 예산, 목표 수익률, 최대 손실율 입력한 뒤 예산을 변경하면,
		- 목표 수익률(%)이 변하는데 %가 아니라 금액이 변하도록 수정
		- 최대 손실율 금액도 변하도록 수정
	- 예산 입력했지만 목표 수익률 및 최대 손실율을 입력안해도 되게 수정
- [x] 포트폴리오 삭제 후 portfolioList query invalidate 필요
- [x] 포트폴리오 삭제 Dialog
	- "항목을" -> "포트폴리오를"로 변경
- [ ] 포트폴리오 상세 페이지
	- 목표 수익률 및 최대 손실율 알림 활성화 버튼 추가
	- "잠정 손실잔고", "투자대비 연 배당률" 툴팁 추가
- [ ] 매입이력 추가 테이블에서 메모부분 trim() 때문에 스페이스 안되는듯
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
	- [ ] FCM token 삭제 확인 (NotificationSettingsDialog)
	- [ ] 배포 환경에서 serviceworker 안돌아감
	- [ ] UserContext 및 FCM Token 관련 리팩토링 (재렌더에 의한 불필요한 setupFCMToken 요청)
	- [ ] FCM token 오류 확인 필요
		- 종종 FCM으로부터 발급 받은 토큰이 `404 UNREGISTERED` 오류가 날 때 해당 토큰을 제거하고 새로운 토큰을 발급받아야 함.
		- https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode
- [ ] 로그인 페이지 "내 정보 기억하기" 구현
- [ ] IconButton, TextButton 구현되면 없을 때 구현한 부분 공용 컴포넌트로 교체하기
- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
- [ ] GIthub PR bot 생각해보기
	- 릴리즈, 메인 배포시 PR 내용을 간단하게 요약해서 pr 만들어줄 수 있는 봇?
	- PR 내용으로 올라가는 이슈 번호 나열같은 기능
- [ ] TanStack Query
	- [ ] Query Key 정리
	- [ ] `invalidateQueries()` 정리
- [x] Percentage 값들 실수형으로 변경
	-  백엔드에서 소수 2자리 까지 보내주고 있는 부분 이상 없음 확인 완료
- 마이너스인 경우 화폐단위 앞으로 "-".
- **"안되는" 포트폴리오(ErrorBoundary component가 적용 됨)에서 "되는" 포트폴리오로 이동할 때 holdings 패널이 그대로 ErrorBoundary component가 남아있음. "새로고침" 버튼을 눌러야 갱신 됨.**
- [ ] 포트폴리오 목록 페이지에서 포트폴리오 삭제 시 화면 재렌더링 안됨

- [ ] 알림 설정 토글 브라우저별로(사파리 파이엎폭스 크롬) 테스트 해보기
#### UI
- [ ] Table header column title (Window/MacOS Font 확인 필요)
	- Table header column의 가로 사이즈 문제인듯?
- [ ] "계정 설정" 탭은 이메일/비밀번호 계정일 때만 보이도록 수정 (BE 협의 필요)
	- [ ] User 객체에 OAuth 및 이메일/비밀번호 가입 구분 필요
- [ ] 예상 월 배당금 그래프
	- [ ] 예상 월 배당금 막대 그래프 0원인 경우 hover시 tooltip 안나오게하기
	- [ ] 1000단위 "," 표시
- [ ] `/src/assets/icons/logo/ic_fineants.svg` 해당 경로 svg 사이즈 조절이 필요해 보임

#### 기타
- `Third-party cookie will be blocked. Learn more in the Issues tab.`
- [ ] Mobile(태블릿 포함) 화면을 위한 임시 안내문 (모달).
	- media query (1200px?)
- [ ] input 내에 숫자 1천 단위 `,`  표시해주는 util 함수 구현
- [ ] `@fineants/demolition`
	- [ ] Contribution 관련 지침
- [ ] 포트폴리오 상세 페이지 "총 손익", "당일 손익" 실시간 변동 시 색상 적용

#### BE
- [ ] User 객체에 OAuth 및 이메일/비밀번호 가입 구분 필요
