# TODO

- [ ] **Landing Page**
	- [ ] 디자인은 없지만 BasePage 적용해 놓기

- [ ] **Dashboard Page**
	- [ ] 포트폴리오 PieChart 예산이 아니라 평가금액으로 변경
	- [ ] ValuationOverview 값
	- [ ] "총 자산 추이" 그래프에서 현재 선택된 기간 highlight

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
	- [ ] `useStompWithRQ` 구조 수정 필요
	- [ ] 목표 손익률, 최대 손실률input에 소숫점 2자리 문제
	- [ ] 증권사 로고 추가 하기

- [ ] **Watchlist Page**
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
	    - [ ] msw에 portfolios 값이 react에서 사용처와 달라 에러 나고 있음
	- [ ] Sector
	    - [ ] 하드코딩 제거

- [ ] **Signup Page**
	- [ ] 이메일/비밀번호
	- [ ] Funnel pattern custom hook
	- [ ] TODO 내용 확인하고 해결하기

- [ ] **OAuth Sign In**
	- [ ] Redirect URI
	    - [ ] Redirect URI를 `/signin`이 아니라 따로 sign in loading page(Ex: “로그인 중입니다”)로 구현
	- [ ] GoogleSignInButton
	    - [ ] `@react/oauth-google`은 Client ID를 FE에 명시하는 것을 필요로하기 때문에 사용하지 말고 직접 구현
	    - [ ] `@react/oauth-google`을 사용한다면 Custom login button (`useGoogleLogin` "auth-code" flow)
	    - [ ] Handle error from Google
	- [ ] KakaoSignInButton
	    - [ ] Handle case where popup doesn't show

- [ ] **Modal 컴포넌트**
	- [ ] `Modal` -> `Dialog`
	    - [ ] "Modal" refers to an element that blocks interaction with the rest of the application.
	- [ ] `ConfirmAlert` -> `ConfirmDialog`
	    - [ ] "Alert" displays a short, important message to the user without interrupting the user's task.

- [ ] **Portfolio Modal**
	- [ ] 무슨 리팩토링이 시급했는지 찾기
	- [ ] 수정, 추가 로직이 추가 되었으니 TODO 지우기
	- [ ] Button disabled 조건식 추가하기
	- [ ] Input 소수점 2자리 문제 해결하기

- [ ] **TotalValuationLineChart**
	- [ ] TODO 주석 2개 해결하기

- [ ] **RechartPieChart**
	- [ ] api 추가
	- [ ] 참고용 주석 따로 보관하기

- [ ] **Footer 컴포넌트**
	- [ ] @ -> ©

- [ ] **portfolioData**
	- [ ] TODO 주석 확인하고 해결하기

- [ ] **기타**
	- [ ] 금액 1,000단위로 comma 찍기
	- [ ] 화폐단위 KRW
	- [ ] 계산 test code
	- [ ] 공용 컴포넌트
	    - [ ] toast
	    - [ ] error component
	    - [ ] loading component
	    - [ ] 아이콘
	- [ ] 각 api 호출에 맞는 error handle

- [ ] **Docs Todo**
	- [ ] FE Architecture
	    - Amplify, React Query, HTTPS, WSS
	- [ ] State Management