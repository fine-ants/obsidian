# TODO

- [ ] **Landing Page**
	- [ ] Figma 디자인 적용
	- [ ] /api/portfolios으로 GET 요청 보내지고 있는 문제

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

- [ ] **Signup Page**
	- [ ] 이메일/비밀번호 가입
	- [ ] 회원가입 페이지 디자인에 맞게 단계 구성

- [ ] 공용 컴포넌트
	- [ ] Button
		- [ ] Refactoring
			- [ ] Button 컴포넌트 `"text"` variant 스타일 확인 필요
		- [ ] 아이콘 버튼
	- [ ] Icon
		- [ ] Hover 필요한 icon 사용처 수정 (`hoverColor` prop 사용)

- [ ] 배포
	- [ ] Custom domain name (fineants) 적용. Docs 참고.

- [ ] **개별 종목 현재가 알림**

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
	- [ ] Layout Shift
		- [ ] SignInPage image layout shift

- [ ] **WatchlistsPage 및 WatchlistPage**

- [ ] Proprietary License

- [ ] Received Dividends Record Feature

- [ ] PortfoliosDropdown 누를 시 stale 값으로 먼저 렌더(포트폴리오 이동, 포트폴리오 추가)된 후 포트폴리오 목록 렌더로 인한 layout/repaint 일어남.

- [ ] 만료된 Refresh Token인 상태로 첫 페이지 로드시, "토큰이 존재하지 않습니다" 토스트 발생. <-- (현재 배포 상황. 확인 필요.)
	- Refresh Token이 만료된 응답이 오면 localStorage에 있는 `user` 초기화 및 signin page으로 이동.

- BE
	- 대시보드 오버뷰 API 
		- totalGain 필드 누락

### 1/19 배포 목표
- [ ] 프로필 페이지 (카카모토비) - API 대기
	- [x] `Header`에 있는 `SearchBar`에 search term highlight이 적용 안됨.
	- [x] `UserDropdown`에 있는 `img`에 image가 없을시 (alt 표시) size 조정 필요.
	- [ ] `user` 객체 `profileUrl` 값
		- http://k.kakaocdn.net/dn/dpk9l1/btqmGhA2lKL/Oz0wDuJn1YV2DIn92f6DVK/img_640x640.jpg <-- ?
	- [x] 프로필 설정
		- [x] 프로필 정보 변경 MSW
	- [x] 계정 설정
		- [x] "계정 삭제하기" dialog
		- [x] New Password MSW
		- [x] Account Delete MSW
	- [ ] Overview data
		- `username` field? 대신 user 객체 사용?
	- [x] SearchBar dropdown 빈문자열일 때 숨김
	- [x] Nickname 중복 확인 로직 hook으로 분리 (`ProfileSettingsSubPage`, `NicknameSubPage`)
- [x] MSW 1.x --> 2.x migration
	- https://mswjs.io/docs/migrations/1.x-to-2.x
- [x] `@fineants/demolition` (카카모토비)
	- [x] `useText`의 값이 빈문자열일때는 `isError`가 `false`이 되도록 수정.
	- [x] `useImageInput`
		- [x] `error` 정상 작동 확인 필요.
		- [x] `onClearImage` 함수 추가
- [ ] 관심목록 (박하)
	- [ ] 관심종목 단일목록 조회시 이름 필요
	- [ ] `StockPage`에서 로그인 되었을 때만 "관심 종목 해제/추가" 버튼 표시
- [ ] Table 관련 수정 (박하)
	- [ ] `TablePagination`의 `rowsPerPageOptions` dropdown height 조정.
	- [ ] `Select`의 `MenuProps`의 `minHeight` 수정 필요.
		- 포트폴리오 추가 모달 부분에서 Select의 dropdown item 스타일 Figma design 적용.
- [ ] 이메일/회원가입 (제이) - API 대기
	- [ ] 로그인 페이지 이메일 input에 text가 있을 시 endAdornment 상시 표시로 수정
- [ ] Design System Font에 letter-spacing 적용 (박하) - Watchlist API 완성 후 PR 머지
- 기타
	- [ ] Toast Design 적용 (Jay) - Design 대기
	- [ ] 예산 0 가능하게, 목표 수익률 손실률도 설정안해도 가능하게 - 진행 가능
	``

### QA
- [x] `PortfolioPage`에 `PortfolioHoldingRow` 개수 font 수정
- [x] `PortfolioListTable` 빈칸 `height` 조정 필요.
- [x] 종목 테이블 "연 배당금" 헤더 hover 시 tooltip ("예상 연 배당금")
- [x] 포트폴리오 상세페이지에서 삭제 후 이동이 안됨.
	- 포트폴리오 목록 페이지로 이동해야 함.

- Portfolios Dropdown 데이터가 오면 dropdown 전체 재렌더됨.

- 매입 이력 추가했는데 현금이 부족 하면 400 Bad Request (사용자 입장에서는 아무 일이 안일어남)
- 매입 이력 추가시 해당 종목의 row의 평가금액이 update 안됨 (아마 정적 데이터 fetch에 대한 invalidateQuery 필요).

- 매입 이력 수정 DatePicker
	- 매입 이력 수정 모드일 때 우리 DatePicker로 교체.
	- 초기값을 오늘 날짜말고 매입이력 등록된 날짜 적용.

- Charts
	- Chart Legend 패딩 및 높이 조절 필요
	- TallChartLegend에 "기타"가 없으면 divider 미표시.

- 이메일 회원가입
	- 비밀번호 생성 단계(2 단계)에서 "다음" 누르면 닉네임 단계 넘기고 프로필 이미지 등록 단계(4단계)로 가짐.

- Indices Page
	- TradingView widget이 자동으로 dark mode됨.

- SearchBar Placeholder
	- "검색어를 입력하세요" --> "종목, 지수를 검색하세요"?

- To Backend
	- PortfolioPage 당일손익금 및 당일손익률 데이터 확인 필요.
	- 차트 데이터 정렬 부탁.
		- DashboardPage 포트폴리오 비중 
		- PortfolioPage 종목 구성 차트
		- SectorBar
	- 배당금 데이터

- To Design
	- Dark Mode

- [ ] Use PasswordTextField's `helperText` instead of `TextInputError` (`SignInForm`)