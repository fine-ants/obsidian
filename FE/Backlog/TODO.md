# TODO

- [ ] **Dashboard Page**
	- [ ] 포트폴리오 비중 파이차트 슬라이스 순서 수정
	- [ ] SSE 적용

- [ ] **Portfolio Page**
	- [ ] Overview 최대 손실율에 대한 논의
	- [ ] PortfolioHoldingsPieChart 실시간으로 변경
	- [ ] 포트폴리오 상세 조회 및 종목 조회
		- `portfolioDetails`에서 실시간 항목이 6개.
		- `portfolioHoldings`에서 실시간 항목이 6개.
		- 나머지 정적인 데이터들을 SSE로 계속 받는 것에 대한 overhead 확인 필요.
		- 문제: 매입 이력을 추가하는 등 CRUD 작업을 진행하면 다음 SSE message을 기다려야함.
		- CRUD vs 실시간 분리.
		- 현재 상황에서는 서버에서 부담스러움
		- 대안
			- REST API로 우선 화면 초기화.
			- 이후 SSE로 실시간 데이터만 받아서 화면 업데이트.
			- CUD 요청이 생길 시, SSE 연결 끊고, 1번, 2번 반복.

- [ ] 공용 컴포넌트
	- [ ] Button
		- [ ] Refactoring
			- [ ] Button 컴포넌트 `"text"` variant 스타일 확인 필요
		- [ ] 아이콘 버튼
	- [ ] Icon
		- [ ] Hover 필요한 icon 사용처 수정 (`hoverColor` prop 사용)

- [ ] 배포
	- [ ] Custom domain name (fineants) 적용. Docs 참고.

- [ ] **알림 기능**

- [ ] 반응형

- [ ] Test (E2E)

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

- [ ] **WatchlistsPage 및 WatchlistPage**

- [ ] Proprietary License

- [ ] Received Dividends Record Feature

- [ ] 만료된 Refresh Token인 상태로 첫 페이지 로드시, "토큰이 존재하지 않습니다" 토스트 발생. <-- (현재 배포 상황. 확인 필요.)
	- Refresh Token이 만료된 응답이 오면 localStorage에 있는 `user` 초기화 및 signin page으로 이동.

- 매입 이력 추가했는데 현금이 부족 하면 400 Bad Request (사용자 입장에서는 아무 일이 안일어남)
- 매입 이력 추가시 해당 종목의 row의 평가금액이 update 안됨 (아마 정적 데이터 fetch에 대한 invalidateQuery 필요).

- [ ] `user` 객체 `profileUrl` 값
	- http://k.kakaocdn.net/dn/dpk9l1/btqmGhA2lKL/Oz0wDuJn1YV2DIn92f6DVK/img_640x640.jpg <-- ?

- Charts
	- Chart Legend 패딩 및 높이 조절 필요.
	- 실시간 데이터 반영.

- To Backend
	- PortfolioPage 당일손익금 및 당일손익률 데이터 확인 필요.
	- 차트 데이터 정렬 부탁.
		- DashboardPage 포트폴리오 비중 
		- PortfolioPage 종목 구성 차트
		- SectorBar
	- 배당금 데이터
	- 이메일 인증 코드 발송 안됨

- To Design
	- Dark Mode

## 2/29 배포 목표
### Kakamotobi & Jay
- [ ] `@fineants/demolition`
	- [ ] Contribution 관련 지침
	- [ ] 훅 및 유틸 함수 추가
		- [ ] formatTickValue <-- 업그레이드 필요
		- [x] getElapsedSince <-- 업그레이드 필요 (Jay)
		- [x] retryFn <-- 향상해보기 (Kakamotobi)
	- [ ] Tree-shaking 도입
### Kakamotobi
- [ ] FCM service worker 문제 해결
- [ ] 잠정 손실잔고, 등 "?" helper 추가
- [ ] 최신 @fineants/demolition package 적용
- [ ] README update (알림 관련 내용 추가)
### Jay
- [x] DashboardOverview 예상 연배당률 (overviewData.totalAnnualDividendYield)에 RateBadge의 화살표가 없어야 함.
- [x] Stock Page "알림 설정"의 "추가된 알림" 지정가 알림 추가 후 invalidate 필요
- [x] 포트폴리오 상세 페이지 "종목 구성 차트 레전드" 내부 아이템 사이즈 UI 수정
	- 폰트 관련 문제와 연관도 있을 것 같아 폰트 문제와 함께 해결하기
- [x] 알림 패널 api 명세와 다른 부분 수정
- [x]  "계정 삭제하기" 줄바꿈 됨
- [x] font 적용 제대로 안된 부분 
- [x] svgr 사용 여부 체크하기

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
#### 기타
- `Third-party cookie will be blocked. Learn more in the Issues tab.`
- [ ] Mobile(태블릿 포함) 화면을 위한 임시 안내문 (모달).
	- media query (1200px?)

- [ ] input 내에 숫자 1천 단위 `,`  표시해주는 util 함수 구현



### TODO
- [ ] FCM
	- [ ] FCM token 삭제 확인 (NotificationSettingsDialog)
	- [ ] 배포 환경에서 serviceworker 안돌아감
	- [ ] UserContext 및 FCM Token 관련 리팩토링 (재렌더에 의한 불필요한 setupFCMToken 요청)
	- [ ] FCM token 오류 확인 필요
		- 종종 FCM으로부터 발급 받은 토큰이 `404 UNREGISTERED` 오류가 날 때 해당 토큰을 제거하고 새로운 토큰을 발급받아야 함.
		- https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode
- [ ] 로그인 페이지 "내 정보 기억하기" 구현
- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
- [ ] TanStack Query
	- [ ] Query Key 정리
	- [ ] `invalidateQueries()` 정리
- [x] Percentage 값들 실수형으로 변경
	-  백엔드에서 소수 2자리 까지 보내주고 있는 부분 이상 없음 확인 완료


- [ ] 알림 설정 토글 브라우저별로(사파리 파이엎폭스 크롬) 테스트 해보기
#### UI
- [ ] Table header column title (Window/MacOS Font 확인 필요)
	- Table header column의 가로 사이즈 문제인듯?
- [ ] "계정 설정" 탭은 이메일/비밀번호 계정일 때만 보이도록 수정 (BE 협의 필요)
	- [ ] User 객체에 OAuth 및 이메일/비밀번호 가입 구분 필요
- [ ]  예상 월 배당금 막대 그래프 0원인 경우 hover시 tooltip 안나오게하기
- [ ] `/src/assets/icons/logo/ic_fineants.svg` 해당 경로 svg 사이즈 조절이 필요해 보임

#### BE
- [ ] User 객체에 OAuth 및 이메일/비밀번호 가입 구분 필요

- To BE
	- 회원가입 인증 코드가 `000006`만 오는 듯함
	- 2-10자를 벗어날 때 client에서 요청을 안보내긴할거지만 중복체크 요청할 때 서버에서 2-10자 검증 필요
	- 로그인이 안되었을 때도 종목 검색 가능하도록 token protection 제거 (BE)