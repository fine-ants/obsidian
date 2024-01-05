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
	- [ ] 실시간 변경 값 1초마다 상승 하락 플래그 다른색깔로 깜빡
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

- [ ] **Profile Page**
	- [x] 스타일
	- [ ] api 추가
	- [ ] 이미지 용량 제한
	- [ ] 예산 0 가능하게, 목표 수익률 손실률도 설정안해도 가능하게

- [ ] **Signup Page**
	- [ ] 이메일/비밀번호 가입
	- [ ] 회원가입 페이지 디자인에 맞게 추가 작업(메일 인증, 프로필 이미지 등록, 현재 가입 단계 표시)

- [x] **SignInLoadingPage**

- [x] **OAuth Sign In**
  - [x] Redirect URI
	- [x] Redirect URI를 `/signin`이 아니라 따로 sign in loading page(Ex: “로그인 중입니다”)로 구현
  - [x] Error 처리
  - [x] Pop Up 흐름
	- [x] Chrome, Safari, Firefox
	  - 문제: 팝업 미허용인 상태일 때 로그인 시도시, 로그인을 계속 진행하는데 문제 생김 (window.opener가 null인데 진행하려는 상태).
	  - 대안: 팝업 열기 시도 전 팝업 허용 상태 확인 후, 미허용인 상태면 팝업을 먼저 허용해달라는 안내 표시.

- [ ] Header Refactor
	- [ ] NavBar
	- [ ] SearchBar
	- [ ] UserControls

- [ ] 공용 컴포넌트
	- [ ] Button
		- [ ] Refactoring
			- [ ] Button 컴포넌트 `"text"` variant 스타일 확인 필요
		- [ ] 아이콘 버튼
	- [ ] Scroll
	- [x] Select
		- [ ] 168px 기준으로 Scroll 적용
	- [ ] SearchBar refactor
	- [x] TextField
	- [x] Tab 컴포넌트화
	- [x] Breadcrumb
	- [ ] Icon
		- [ ] Hover 색상 처리

- [ ] Router 관련 수정
	- 지금 각 page에 header가 붙어 있어서 route가 진행될 때 마다 header가 새로 랜더링되고 있어서 깜빡거립니다 (TVTickerTapWidget).

- [ ] 배포
	- [ ] Custom domain name (fineants) 적용. Docs 참고.

- [ ] README.md
	- [ ] Tech stack, links, etc.

- [ ] 개별 종목 현재가 알림

- [x] Pie Chart type에서 `fill` 제거 (프론트에서 색상 핸들링)

- [ ] 반응형

- [ ] `@fineants/demolition`
	- [x] 현재 있는 hook 및 util library로 이동
	- [x] `useInputDebounce` test code 작성
	- [x] JSDocs 작성
	- [ ] README 작성
	- [ ] Open Source

- [x] Toast 사용 줄임
	- [x] 검색 에러 토스트 제거
	- [x] 조회 실패 토스트 제거
	- [x] Toast 위치 변경 (중간?)

- [ ] **기타**
	- [ ] 화폐단위 KRW
	- [x] 사용하지 않는 이미지, 아이콘 지우기
	- [x] Header에 로고 클릭시 로그인 상태 비교해서 랜딩 or 대시보드
	- [x] refetchOnWindowFocus 옵션 제거

- [ ] Docs ErrorBoundary 관련 내용 추가
	- `componentDidCatch(error, info)` api를 통해 render 과정에서 일어나는 에러를 잡을 수 있는데, 이는 현재 Class component에만 available하다. ErrorBoundary class component을 만들거나 `react-error-boundary` library를 사용.
		- https://react.dev/reference/react/Component#componentdidcatch-caveats
		- https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary
		- https://react.dev/reference/react/Component#componentdidcatch
	- `useSuspenseQuery` relies on the internal cache for suspense functionality, and the cache operates independently of the `gcTime` configuration. Therefore, error responses are mandatorily cached despite manually setting `gcTime: 0`.
	- When using **suspense** or **throwOnError** in your queries, you need a way to let queries know that you want to try again when re-rendering after some error occurred. With the `QueryErrorResetBoundary` component you can reset any query errors within the boundaries of the component.
	- [ ] https://tanstack.com/query/v5/docs/react/reference/QueryErrorResetBoundary

- [x] PortfolioListTable
	- [x] rowsPerPageOptions All로 했을 때 sorting 적용안됨.
	- [x] rowsPerPageOptions -1을 All로 display해야 됨.

- 우선순위
	- Suspense, ErrorBoundary
		- [ ] 포트폴리오 상세 페이지 (박하)
		- [x] 포트폴리오 목록 페이지 (카카)
		- [x] 글로벌(루트 레벨) 처리 (카카)
		- [x] 관심 목록 페이지 (카카)
	- [ ] PortfolioHoldingsPieChart 실시간으로 변경

- [ ] 포트폴리오 Q/A
	- [ ] 포트폴리오 종목 상세 페이지에서 아코디언이 펼쳐져 있는 경우 header에 수정, 삭제 아이콘 삭제하기
	- [x] 포트폴리오 아코디언에 있는 가로, 아래 화살표 크기 문제 확인하고 수정하기
	- [ ] 포트폴리오 수정, 추가 버튼에 아이콘 추가하기
	- [ ] 종목 삭제 버튼에 delete 아이콘 추가하기
	- [x] 종목 추가 dialog 종목 검색 해서 추가 되었을 경우 dialog 높이 변하게 추가(피그마에 나온 것 처럼)
	- [ ] 종목 추가 dialog에 우측 상단 닫기 버튼 추가
	- [ ] 종목 추가 버튼 디자인과 동일하게 수정하기(디자인에는 + 아이콘있고, border가 없음)

- [ ] Optimization
	- [ ] Performance
		- [ ] FCP, LCP
	- [ ] Accessibility
		- [ ] Navigate search result dropdown using arrow keys
		- [ ] Button (https://dequeuniversity.com/rules/axe/4.7/button-name)
		- [ ] Image (https://dequeuniversity.com/rules/axe/4.7/image-alt)

- [ ] Table 관련 컴포넌트 refactor

- [ ] **WatchlistsPage 및 WatchlistPage**

- [ ] Proprietary License

- [ ] Received Dividends Record Feature

- BE
	- 증권사 로고 한글로 변경 필요 (`securitiesFirm`)
	- WatchlistItemType에 `dateAdded`, `dailyChange` field 추가
