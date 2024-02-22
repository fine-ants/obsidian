# TODO


- [ ] **Dashboard Page**
	- [ ] 대시보드 오버뷰 API totalGain 필드 누락됨 (BE)
	- [ ] 포트폴리오 비중 파이차트 슬라이스 순서 수정
	- [ ] SSE 적용

- [ ] **Portfolio Page**
	- [ ] Overview 최대 손실율에 대한 논의
	- [ ] Prevent adding duplicate 종목
		- [ ] 에러메세지 수정 필요
	- [ ] 연배당률 소수점 포함 (BE)
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

- [ ] `@fineants/demolition`
	- [ ] README 작성
	- [ ] Open Source

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
	- TallChartLegend에 "기타"가 없으면 divider 미표시.
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

### 2/23 배포 목표
#### Kakamotobi
- [x] FCM token delete request
- [x] 종목 페이지 알림 설정 dropdown 추가된 알림 부분
- [x] 알림 설정 토글 초기값 확인 (다 false로 하고 저장을 했는데 toggle은 true로 되어있음)
- [x] StockPage alert dialog에서 지정가 등록하고 targetprices invalidate 필요.
- [ ] Domain 옮기기
- [ ] Collapsible Table row expand할 시에 horizontal scroll 생김
- [ ] Table header column title 줄바꿈 됨
#### Jay
- [x] ~~Main application에 "message" listener 처리 및 Foreground 알림 UI
	- -> new Notification()으로 변경
- [x] NotificationPanel "알림이 없습니다" 문구 색상 변경
#### Bakha
- [ ] Watchlist breadcrumb 이름 수정

#### TODO
- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
- [ ] Watchlist에 현재가 조회가 안되는 종목을 추가했을 때 문제 (BE)
- [ ] FCM token 오류 확인 필요
	- 종종 FCM으로부터 발급 받은 토큰이 `404 UNREGISTERED` 오류가 날 때 해당 토큰을 제거하고 새로운 토큰을 발급받아야 함.
	- https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode
- [ ] Query Key 정리
- [ ] `invalidateQueries()` 정리
- [ ] Watchlist 선택 후 삭제 시 table head checkbox deselect 안됨
- [ ] Portfolio List Page `currentValuation` 누락됨 (BE)
- [ ] Portfolio List Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [ ] Watchlist Page `main` height 조정 필요 (scroll이 필요할 때만 되게)
- [ ] FineAnts logo 이미지 사이즈 조정 필요 (40x40인 원안에 들어가도 자연스럽게)
- [ ] 새 리스트 추가 모달 "추가" 버튼 disabled 적용.
- [ ] 포트폴리오 상세 페이지 "종목 추가 모달"에서 매입가 및 매입 개수를 기입했지만 서버에 반영이 안됨
- [ ] 포트폴리오 상세 페이지 "종목 구성 차트 레전드" 내부 아이템 사이즈 UI 수정
- [ ] 포트폴리오 상세 페이지 `main` 부분 (테이블 밑) height 수정
- [ ] 포트폴리오 상세 페이지  예상 월 배당금 막대그래프 호버시 tooltip 2024-NaN 해결
- [ ] 포트폴리오 상세 페이지  섹터 구성 합계가 100%가 아님
- [ ] 포트폴리오 상세 페이지  우측 구성 border-radius 없음

