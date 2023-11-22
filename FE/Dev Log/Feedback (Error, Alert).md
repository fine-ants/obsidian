
# Feedback (Error, Alert)

## Table of Contents
- [[#User Feedbacks]]
- [[#에러 및 피드백 핸들링 방식]]


## User Feedbacks
- CRUD 성공/실패 - Toast
	- Portfolio Page
		- Portfolio 추가, 수정, 삭제
		- Portfolio Holding 추가, 수정, 삭제
	- Watchlist Page
		- Watchlist
			- 추가, 수정 (이름 변경), 삭제
		- Watchlist Stock 추가, 삭제
	- Signup Page
		- 일반 회원가입
	- Signin Page
		- 로그인 실패
	- My Profile Page
		- 프로필 정보 변경
- NotFound Page
- Notification - Toast (manual close)
	- 목표 수익률 도달
	- 최대 손실율 도달

## 에러 및 피드백 핸들링 방식
- Query/Mutation Cache에서 일괄처리
	- 각 query/mutation에서 meta 필드 이용해서 error message 설정
- Suspense
- ErrorBoundary
