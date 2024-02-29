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

## 2/29 배포 목표
### Kakamotobi
- [ ] Watchlist 선택 후 삭제 시 table head checkbox deselect 안됨
- [ ] 새 리스트 추가 모달 "추가" 버튼 disabled 적용
- [ ] "종목 추가 모달"에서 "추가" 버튼 누른 후 spinner 적용
#### Signup Page
- [x] 닉네임 2-10자  일때만 중복체크 요청하도록 수정
- [ ] 프로필 이미지 등록
	- [x] 사이즈 2MB 초과시 에러 문구 한국어로 변경
	- [ ] 사진 첨부 후 "등록 완료" 버튼을 누를 시 400 에러 (Required request part 'signupData' is not present)
	- [ ] 사진 첨부 후 "지금은 건너뛰기" 누르면 400 에러 (Required request part 'signupData' is not present)
		- 사진 첨부안하고 "지금은 건너뛰기" 눌렀을 때는 정상 작동함
- [ ] Enter로 다음 단계로 넘어갈 수 있도록 하기
- To BE
	- 회원가입 인증 코드가 `000006`만 오는 듯함
	- 2-10자를 벗어날 때 client에서 요청을 안보내긴할거지만 중복체크 요청할 때 서버에서 2-10자 검증 필요
#### Profile Settings Page
- 프로필 이미지 "기본 이미지 사용" 후 저장할 때 400에러("변경할 회원 정보가 없습니다") 뜸
- 닉네임 input 비어있을 때 "저장" 버튼 비활성화하기
- 계정 삭제 실패했는데 signout 되는 현상
### Jay
- [x] 포트폴리오 상세 페이지 "종목 추가 모달"에서 `purchaseHistory` 항목이 누락된 채로 요청이 됨
- [x] 포트폴리오 상세 페이지  예상 월 배당금 막대그래프 호버시 tooltip 2024-NaN 해결 (MSW)
- [x] 포트폴리오 상세 페이지  섹터 구성 합계가 100%가 아님 (MSW)
- [x] 포트폴리오 상세 페이지  우측 구성 border-radius 없음
- [ ] 포트폴리오 상세 페이지 `main` 부분 (테이블 밑) height 수정
	- 박하가 해결
#### 활성 알림 페이지
- 종목 알림
	- `notificationPreferences`가 모두 false일 때 안내 문구 ("알림 설정이 비활성화 되어있습니다") 보여주기
	- 알림 활성화/비활성화 버튼을 누를 시 서버에 반영이 되면서 list의 순서가 오래전에 업데이트된 순서로 받아와서 UI shift가 생김
- 포트폴리오 알림
	- 포트폴리오 이름 왼쪽에 증권사 이미지 추가
	- 알림 활성화/비활성화 버튼을 누를 시 서버에 반영이 되면서 list의 순서가 최근 업데이트된 순서로 받아와서 UI shift가 생김
#### 알림
- 첫 회원가입 시 notificationPreferences 모두 false로 설정.
- 알림 설정 Dialog
	- 설정 변경 성공 및 실패시 토스트 띄우기.
- 알림 패널에서 내용이 없을 시 PATCH 모두 읽음 API 호출 안하기.
#### HomePage
- "포트폴리오로 이동"을 한 뒤 뒤로가기하면 homepage로 안돌아가고 signinpage에 머무르는 현상.
- 로그인이 안됐을 때는 "포트폴리오 추가"를 클릭하면 signinpage로 이동해야 함 (현재는 모달을 띄우고 있음).
- 주식 검색
	- 로그인이 안되었을 때도 검색 가능하도록 token protection 제거
	- 로그인이 안되었을 때, StockPage에서 "관심 종목 설정", "알림 설정" 버튼 누를시 SigninPage로 이동해야 함.
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
- [ ] 매입 항목 추가시 인벨리데이트 안됨
#### Watchlist
- Watchlist Table 디자인 수정
	- 별 제거
	- *Watchlist Table 단일 삭제 API 불필요*
- Watchlist 변동률 정렬이 적용이 안됨
#### 기타
- Access token 만료 시 `401`에러가 뜨는지 토스트에 반영이 되는지 확인 및 방지.
	- 비고: 401 뜨고 해당 요청 정상 진행 되면 실패 토스트 다음에 성공 토스트가 뜨는지 확인.
- `Third-party cookie will be blocked. Learn more in the Issues tab.`
- Percentage 소수점 둘째자리까지 보이도록.



### TODO
- [ ] Mobile(태블릿 포함) 화면은 임시 안내문.
	- media query (1200px?)
- [ ] 포트폴리오 상세 페이지 "종목 구성 차트 레전드" 내부 아이템 사이즈 UI 수정
	- 폰트 관련 문제와 연관도 있을 것 같아 폰트 문제와 함께 해결하기
- [ ] input 내에 숫자 1천 단위 `,`  표시해주는 util 함수 구현
- [ ] Stock Page "알림 설정"의 "추가된 알림" 지정가 알림 추가 후 invalidate 필요
#### Feature
- [ ] Push Service Queue된 메시지 고려 (`install` event 필요할 수도)
- [ ] Query Key 정리
- [ ] `invalidateQueries()` 정리
- [ ] FCM token 오류 확인 필요
	- 종종 FCM으로부터 발급 받은 토큰이 `404 UNREGISTERED` 오류가 날 때 해당 토큰을 제거하고 새로운 토큰을 발급받아야 함.
	- https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode
#### UI
- [ ] Table header column title, "계정 삭제하기" 줄바꿈 됨 (Window/MacOS Font 확인 필요)

- [ ] "계정 설정" 탭은 이메일/비밀번호 계정일 때만 보이도록 수정 (BE 협의 필요)
	- [ ] User 객체에 OAuth 및 이메일/비밀번호 가입 구분 필요\
#### BE
- [ ] 계정 삭제하기 500에러
	- 추측: 500에러나지만 refresh token이 서버에서는 삭제가 되는 듯함.
- [ ] User 객체에 OAuth 및 이메일/비밀번호 가입 구분 필요
