# Branch Strategy

- `main` - 배포
	- 필요시 `hotfix`
- `release` - 배포 테스트
	- 필요시 `main`으로부터 동기화
- `dev` - 개발
	- 필요시 `main`으로부터 동기화
- `feat`/`refactor`/etc - feature branch


`feat` -> `dev` -> `release` -> `main`

