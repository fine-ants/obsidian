# TODO

- [x] **Landing Page**
	- [x] 디자인은 없지만 BasePage 적용해 놓기
	- [ ] /api/portfolios으로 GET 요청 보내지고 있는 문제

- [x] **Dashboard Page**
	- [x] 포트폴리오 리스트 데이터 예산이 아니라 평가금액으로 변경
	- [x] 포트폴리오 파이차트 API 따로 생성
	- [x] ValuationOverview 값
	- [x] "총 자산 추이" 그래프에서 현재 선택된 기간 highlight

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
	- [ ] stomp 웹소켓 -> SSE 변경
	- [ ] 실시간 변경 값 1초마다 상승 하락 플래그 다른색깔로 깜빡

- [x] **Watchlist Page**
	- [x] Hardcode된 포트폴리오 제거
	- [x] Watchlist 추가, 삭제 기능
		- [x] 종목 추가, 삭제 기능
	- [x] DnD library 바꾸기
		- [x] react-movable로 변경

- [ ] **Profile Page**
	- [ ] “내 프로필” 탭
	    - [ ] 스타일
	    - [ ] api 추가
	    - [ ] 이미지 용량 제한
	    - [ ] isNicknameChecked 상태에 맞는 화면 렌더링
	- [ ] “내 포트폴리오” 탭
	    - [ ] 스타일
	    - [ ] 하드코딩 제거
	    - [x] msw에 portfolios 값이 react에서 사용처와 달라 에러 나고 있음
		    - [x] undefined 예외처리해줘서 해결
	- [ ] Sector
	    - [ ] 하드코딩 제거

- [ ] **Signup Page**
	- [ ] 이메일/비밀번호 가입
		- [ ] 이미지 업로드 (S3 presigned URL)
	- [x] Funnel pattern custom hook
	- [x] TODO 내용 확인하고 해결하기

- [ ] **OAuth Sign In**
	- [ ] Redirect URI
	    - [ ] Redirect URI를 `/signin`이 아니라 따로 sign in loading page(Ex: “로그인 중입니다”)로 구현
	- [ ] Google Sign In
	    - [ ] `@react/oauth-google`은 Client ID를 FE에 명시하는 것을 필요로하기 때문에 사용하지 말고 직접 구현
	    - [ ] `@react/oauth-google`을 사용한다면 Custom login button (`useGoogleLogin` "auth-code" flow)
	    - [ ] Handle error from Google
	- [x] Kakao Sign In
	    - [x] 대안3으로 refactor
	- [x] Naver Sign In
		- [x] Backend로부터 Authorization URL 받는 방식으로 refactor
	- [ ] Pop Up 흐름
		- [ ] Chrome, Safari, Firefox
			- 문제: 팝업 미허용인 상태일 때 로그인 시도시, 로그인을 계속 진행하는데 문제 생김 (window.opener가 null인데 진행하려는 상태).
			- 대안: 팝업 열기 시도 전 팝업 허용 상태 확인 후, 미허용인 상태면 팝업을 먼저 허용해달라는 안내 표시.
	- [ ] Pop Up Window 종종 original window로 이동이 안됨 (로그인 안됨).

- [x] **Modal 컴포넌트**
	- [x] `Modal` -> `Dialog`
	    - [x] "Modal" refers to an element that blocks interaction with the rest of the application.
	- [x] `ConfirmAlert` -> `ConfirmDialog`
	    - [x] "Alert" displays a short, important message to the user without interrupting the user's task.
	- 해당 TODO Issue : https://github.com/fine-ants/frontend/issues/5

- [x] **Portfolio Dialog**
	- [ ] ~~무슨 리팩토링이 시급했는지 찾기
		- 컴포넌트 사이즈가 커서 추가/수정을 분리로 추정된다.
		- 하지만 분리가 오히려 복잡도가 높아질 것 같아서 분리하지 않았습니다.
	- [x] 수정, 추가 로직이 추가 되었으니 TODO 지우기
	- [x] Button disabled 조건식 추가하기
	- [x] Input 소수점 2자리 문제 해결하기
	- 해당 TODO Issue : https://github.com/fine-ants/frontend/issues/8

- [ ] **TotalValuationLineChart**
	- [ ] TODO 주석 2개 해결하기

- [ ] **RechartPieChart**
	- [ ] api 추가
	- [ ] 참고용 주석 따로 보관하기

- [x] **Footer 컴포넌트**
	- [x] @ -> ©

- [ ] **portfolioData**
	- [ ] TODO 주석 확인하고 해결하기

- [ ] **기타**
	- [ ] 금액 1,000단위로 comma 찍기
	- [ ] 화폐단위 KRW
	- [ ] 계산 test code
	- [ ] 각 api 호출에 맞는 error handle (Query/Mutation Cache, Suspense, etc)

- [ ] 공용 컴포넌트
	- [x] toast
	- [ ] error component
	- [ ] loading component
	- [ ] 아이콘 버튼

- [ ] Router 관련 수정
	-  지금 각 page에 header가 붙어 있어서 route가 진행될 때 마다 header가 새로 랜더링되고 있어서 깜빡거립니다

- [ ] securitiesFirm 로고 이미지 추가하기 

- [ ] 배포
	- [ ] Custom domain name (fineants) 적용.

- [x] Husky 고치기

- [ ] README.md
	- [ ] Tech stack, links, etc.

- [ ] Dropdown component refactor

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
	- 문제: 매입 이력을 추가하는 등 CRUD 작업을 진행하면 
	- CRUD vs 실시간 분리.
	- 현재 테스트 중: SSE 주기 5초 -> 1초

- [ ] 문제: 다른 사용자가 만든 portfolio를 `/portfolio/:portfolioId`로 접근이 가능함.
