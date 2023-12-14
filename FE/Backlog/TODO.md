# TODO

- [x] **Landing Page**
	- [x] 디자인은 없지만 BasePage 적용해 놓기
	- [ ] /api/portfolios으로 GET 요청 보내지고 있는 문제

- [ ] **Portfolio Page**
	- [ ] Overview 부분에 포트폴리오 평가금액 추가
	- [ ] Overview 최대 손실율에 대한 논의
	- [ ] 매입이력
		- [ ] 각 매입이력별 손익률
		- [ ] 매입 이력 추가 시 금액 합계 보이기
	- [ ] Prevent adding duplicate 종목
	- [ ] 배당금 차트
		- [x] 현재월 highlight 되도록 수정
		- [ ] 0일 경우 bar 없음
	- [ ] 연배당률 소수점 포함
	- [x] stomp 웹소켓 -> SSE 변경
	- [ ] 실시간 변경 값 1초마다 상승 하락 플래그 다른색깔로 깜빡
	- [ ] Sector
		- [ ] 하드코딩 제거

- [ ] **Profile Page**
	- [ ] 스타일
	- [ ] api 추가
	- [ ] 이미지 용량 제한

- [ ] **Signup Page**
	- [ ] 이메일/비밀번호 가입
		- [ ] 이미지 업로드 (S3 presigned URL)

- [ ] **OAuth Sign In**
  - [ ] Redirect URI
	- [ ] Redirect URI를 `/signin`이 아니라 따로 sign in loading page(Ex: “로그인 중입니다”)로 구현
  - [ ] Error 처리
  - [ ] Pop Up 흐름
	- [ ] Chrome, Safari, Firefox
	  - 문제: 팝업 미허용인 상태일 때 로그인 시도시, 로그인을 계속 진행하는데 문제 생김 (window.opener가 null인데 진행하려는 상태).
	  - 대안: 팝업 열기 시도 전 팝업 허용 상태 확인 후, 미허용인 상태면 팝업을 먼저 허용해달라는 안내 표시.
  - [ ] Pop Up Window 종종 로그인이 진행이 안됨.
	- Pop Up이 열리고 바로 닫힘.

- [ ] **기타**
	- [ ] 화폐단위 KRW

- [ ] 공용 컴포넌트
	- [ ] 아이콘 버튼

- [ ] Router 관련 수정
	- 지금 각 page에 header가 붙어 있어서 route가 진행될 때 마다 header가 새로 랜더링되고 있어서 깜빡거립니다

- [x] securitiesFirm 로고 이미지 추가하기

- [ ] 배포

  - [ ] Custom domain name (fineants) 적용.

- [x] Husky 고치기

- [ ] README.md

  - [ ] Tech stack, links, etc.

- [x] Dropdown component refactor

- [ ] 개별 종목 현재가 알림

- [ ] Pie Chart type에서 `fill` 제거 (프론트에서 색상 핸들링)

- [ ] Button 컴포넌트 `"text"` variant 스타일 확인 필요

- [ ] Toast

  - [ ] 위치 변경
  - [ ] 종목 검색 시 다수 toast 발생

- [ ] 포트폴리오 상세 조회 및 종목 조회

  - `portfolioDetails`에서 실시간 항목이 6개.
  - `portfolioHoldings`에서 실시간 항목이 6개.
  - 나머지 정적인 데이터들을 SSE로 계속 받는 것에 대한 overhead 확인 필요.
  - 문제: 매입 이력을 추가하는 등 CRUD 작업을 진행하면 다음 SSE message을 기다려야함.
  - CRUD vs 실시간 분리.
  - 고려한 대안: SSE 주기 5초 -> 1초
    - 현재 상황에서는 서버에서 부담스러움
  - 대안
    - REST API로 우선 화면 초기화.
    - 이후 SSE로 실시간 데이터만 받아서 화면 업데이트.
    - CUD 요청이 생길 시, SSE 연결 끊고, 1번, 2번 반복.

- [ ] 문제: 다른 사용자가 만든 portfolio를 `/portfolio/:portfolioId`로 접근이 가능함.

- [ ] DashboardPage 포트폴리오 비중 차트 레전드 스크롤 UI 수정

- [ ] 반응형

- [ ] `useInputDebounce` test code 작성

- [ ] Toast 사용 줄임

  - [ ] 검색 에러 토스트 제거
  - [ ] 조회 실패 토스트 제거

- [ ] 사용하지 않는 이미지, 아이콘 지우기
- [ ] Hooks 및 Utils 제거 후 `@fineants/demolition` 패키지 적용
- [ ] Header에 로고 클릭시 dashboard가 아닌 랜딩 페이지로
- [ ] 회원가입 페이지 디자인에 맞게 추가 작업(메일 인증, 프로필 이미지 등록, 현재 가입 단계 표시)

- 우선순위
  - Search debounce 적용 (카카 & 박하)
  - Suspense, ErrorBoundary
    - 대시보드 페이지 (제이)
    - 포트폴리오 상세 페이지 (박하)
    - 포트폴리오 목록 페이지 (카카)
    - 글로벌(루트 레벨) 처리 (카카)
    - 관심 목록 페이지 (높은 확률로 제이)
  - PortfolioHoldingsPieChart 실시간으로 변경
  - 반응형
