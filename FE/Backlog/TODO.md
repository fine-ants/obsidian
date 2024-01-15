# TODO

- [x] **Landing Page**
	- [x] 디자인은 없지만 BasePage 적용해 놓기
	- [ ] /api/portfolios으로 GET 요청 보내지고 있는 문제

- [ ] **Portfolio Page**
	- [x] Overview 부분에 포트폴리오 평가금액 추가
	- [ ] Overview 최대 손실율에 대한 논의
	- [ ] Prevent adding duplicate 종목
		- [ ] 에러메세지 수정 필요
	- [x] 배당금 차트
		- [x] 현재월 highlight 되도록 수정
		- [x] 0일 경우 bar 없음
	- [ ] 연배당률 소수점 포함
	- [x] stomp 웹소켓 -> SSE 변경
	- [ ] 실시간 변경 값 5초마다 상승 하락 플래그 다른색깔로 깜빡
	- [ ] PortfolioHoldingsPieChart 실시간으로 변경
	- [x] Sector
		- [x] 하드코딩 제거
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
	- [x] Skeleton 구현 필요 (현재 overview skeleton이 보여짐)
	- [ ] 포트폴리오 Q/A
		- [ ] 포트폴리오 종목 상세 페이지에서 아코디언이 펼쳐져 있는 경우 header에 수정, 삭제 아이콘 삭제하기
		- [x] 포트폴리오 아코디언에 있는 가로, 아래 화살표 크기 문제 확인하고 수정하기
		- [ ] 포트폴리오 수정, 추가 버튼에 아이콘 추가하기
		- [ ] 종목 삭제 버튼에 delete 아이콘 추가하기
		- [x] 종목 추가 dialog 종목 검색 해서 추가 되었을 경우 dialog 높이 변하게 추가(피그마에 나온 것 처럼)
		- [ ] 종목 추가 dialog에 우측 상단 닫기 버튼 추가
		- [ ] 종목 추가 버튼 디자인과 동일하게 수정하기(디자인에는 + 아이콘있고, border가 없음)

- [ ] **Profile Page**
	- [x] 스타일
	- [ ] api 추가
	- [ ] 이미지 용량 제한

- [x] **Signup Page**
	- [x] 이메일/비밀번호 가입
	- [x] 회원가입 페이지 디자인에 맞게 추가 작업(메일 인증, 프로필 이미지 등록, 현재 가입 단계 표시)

- [x] **SignInLoadingPage**

- [x] **OAuth Sign In**
  - [x] Redirect URI
	- [x] Redirect URI를 `/signin`이 아니라 따로 sign in loading page(Ex: “로그인 중입니다”)로 구현
  - [x] Error 처리
  - [x] Pop Up 흐름
	- [x] Chrome, Safari, Firefox
	  - 문제: 팝업 미허용인 상태일 때 로그인 시도시, 로그인을 계속 진행하는데 문제 생김 (window.opener가 null인데 진행하려는 상태).
	  - 대안: 팝업 열기 시도 전 팝업 허용 상태 확인 후, 미허용인 상태면 팝업을 먼저 허용해달라는 안내 표시.

- [ ] 공용 컴포넌트
	- [ ] Button
		- [ ] Refactoring
			- [ ] Button 컴포넌트 `"text"` variant 스타일 확인 필요
		- [ ] 아이콘 버튼
	- [x] Scroll
	- [x] Select
		- [ ] 특정 height 기준으로 Scroll 적용
	- [x] SearchBar refactor
	- [x] TextField
	- [x] Tab 컴포넌트화
	- [x] Breadcrumb
	- [ ] Icon
		- [x] Hover 색상 처리
		- [ ] Hover 필요한 icon 사용처 수정 (`hoverColor` prop 사용)

- [ ] 배포
	- [ ] Custom domain name (fineants) 적용. Docs 참고.

- [ ] 개별 종목 현재가 알림

- [ ] 반응형

- [ ] `@fineants/demolition`
	- [ ] README 작성
	- [ ] Open Source

- [ ] **기타**
	- [ ] 화폐단위 KRW

- [ ] Optimization
	- [ ] Performance
		- [ ] FCP, LCP
	- [ ] Accessibility
		- [ ] Navigate search result dropdown using arrow keys
		- [ ] Button (https://dequeuniversity.com/rules/axe/4.7/button-name)
		- [ ] Image (https://dequeuniversity.com/rules/axe/4.7/image-alt)

- [x] Table 관련 컴포넌트 refactor

- [ ] **WatchlistsPage 및 WatchlistPage**

- [ ] Proprietary License

- [ ] Received Dividends Record Feature

- [ ] Use PasswordTextField's `helperText` instead of `TextInputError`

- [ ] PortfoliosDropdown 누를 시 stale 값으로 먼저 렌더(포트폴리오 이동, 포트폴리오 추가)된 후 포트폴리오 목록 렌더로 인한 layout/repaint 일어남.

- [ ] 만료된 Refresh Token인 상태로 첫 페이지 로드시, "토큰이 존재하지 않습니다" 토스트 발생. <-- (현재 배포 상황. 확인 필요.)
	- Refresh Token이 만료된 응답이 오면 localStorage에 있는 `user` 초기화 및 signin page으로 이동.

- BE
	- WatchlistItemType에 `dateAdded`, `dailyChange` field 추가
	- 대시보드 오버뷰 API 
		- totalGain 필드 누락

- 1/12 배포 목표
	- [x] Header Refactor (카카모토비)
		- [x] NavBar
		- [x] SearchBar
		- [x] UserControls
	- [ ] 프로필 페이지 (카카모토비) - Design, API 대기
	- [ ] 관심목록 (박하) - API 대기
		- [ ] 관심종목 단일목록 조회시 이름 필요
	- [ ] 이메일/회원가입 (제이) - API 대기
		- [ ] 로그인 페이지 이메일 input에 text가 있을 시 endAdornment 상시 표시로 수정
	- [x] 포트폴리오 상세 페이지 SSE (박하) - 진행 중
	- [ ] Table Refactor (카카) - PR 올림
	- [ ] Design System Font에 letter-spacing 적용 (박하) - Watchlist API 완성 후 PR 머지
	- [x] Access token을 refresh할 때 error 무한 루프
	- 기타
		- [ ] Toast Design 적용 (Jay) - Design 대기
		- [ ] 예산 0 가능하게, 목표 수익률 손실률도 설정안해도 가능하게 - 진행 가능
		- [x] Router 관련 수정
			- 지금 각 page에 header가 붙어 있어서 route가 진행될 때 마다 header가 새로 랜더링되고 있어서 깜빡거립니다 (TVTickerTapeWidget).

- `PortfolioPage`에 `PortfolioHoldingRow` 개수 font 수정

- `Header`에 있는 `SearchBar`에 search term highlight이 적용 안됨.

