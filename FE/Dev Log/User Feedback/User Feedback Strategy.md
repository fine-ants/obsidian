# User Feedback Strategy

## Table of Contents
- [[#User Feedbacks]]
- [[#에러 및 피드백 핸들링 방식]]

## User Feedbacks
- Toast 처리
	- Mutation (CUD) 관련 성공/실패
		- Ex: Portfolio 추가/수정/삭제, Watchlist 추가/수정/삭제, 로그인 실패, 프로필 정보 변경.
- ErrorBoundary
	- Query (Read) 관련 실패
	- NotFoundPage
- Suspense
	- Skeleton UI 또는 spinner

## 에러 및 피드백 핸들링 방식
- Toast 처리
	- Query/Mutation Cache에서 에러 일괄 처리
	- 각 query/mutation에서 meta 필드 이용해서 `toastErrorMessage`, `toastSuccessMessage` 설정
- ErrorBoundary 및 Suspense
	- User Component에 가장 근접한 곳에서 처리
