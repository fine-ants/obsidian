# TODO

- [ ] **Dashboard Page**
	- [ ] 대시보드 오버뷰 API totalGain 필드 누락됨
	- [ ] SSE 적용

- [ ] **Portfolio Page**
	- [ ] Overview 최대 손실율에 대한 논의
	- [ ] Prevent adding duplicate 종목
		- [ ] 에러메세지 수정 필요
	- [ ] 연배당률 소수점 포함
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
	- [ ] 포트폴리오 종목 상세 페이지에서 아코디언이 펼쳐져 있는 경우 header에 수정, 삭제 아이콘 삭제하기
	- [ ] 포트폴리오 수정, 추가 버튼에 아이콘 추가하기
	- [ ] 종목 삭제 버튼에 delete 아이콘 추가하기
	- [ ] 종목 추가 dialog에 우측 상단 닫기 버튼 추가
	- [ ] 종목 추가 버튼 디자인과 동일하게 수정하기(디자인에는 + 아이콘있고, border가 없음)

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
	- [ ] 관심 종목 현재가 알림 ❌ 종목 현재가 알림 ✅
		- 관심 종목을 기준으로 하면 여러 관심목록에 포함된 종목도 있으니 복잡해짐. 그리고 관심 목록에 들어있는 종목을 다 알림을 받는 것보다 특정한 종목에 대한 알림을 받는게 더 적절함.
		- 종목마다 등록한 특정 가격에 도달하면 알림을 보내기.
	- [ ] 포트폴리오 목표 수익률 및 최대 손실율 도달 알림

- [ ] `@fineants/demolition`
	- [ ] README 작성
	- [ ] Open Source

- [ ] 반응형

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

### 2/16 배포 목표
#### Kakamotobi & Jay
- [ ] 포트폴리오 목표 수익륙/최대 손실율 알림 기능
	- [ ] Service Worker 구현
	- [ ] 알림 설정 Modal API 연동 - Jay
	- [ ] Foreground 알림 UI - Jay
	- [x] FCM Token API 구현 - Kakamotobi
	- [ ] Active Notifications Page API 연동 확인 - Kakamotobi
	- [ ] Main application에 "message" listener 처리 - Kakamotobi
	- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
#### Bakha
- [x] mutation key 제거
- [ ] 기타 사소한 수정 사항
	- [ ] 화폐단위 KRW
