
### act 사용
### act 설치
```shell
brew install act
```

### act 설치 확인
```shell
act --version
```

### 사용 가능한 워크플로우 및 작업 목록
```shell
act -l
```

### 이미지 내려받기
```shell
act pull --container-architecture linux/amd64
```

### 특정 워크플로우 파일 실행
형식
```shell
act -W {workflows 파일 경로}
```

예시
```shell
act -W .github/workflows/ci.cd.production.gcp.yml
```

### 특정 워크플로우 파일의 특정 job 실행
형식
```shell
act -W {workflows 파일 경로} --job {job 이름}
```

예시
```shell
act -W .github/workflows/ci.cd.production.gcp.yml \
--container-architecture linux/amd64 \
--secret-file .secrets \
--bind \
--job build-image
```


```shell
act -W .github/workflows/ci.cd.production.gcp.yml \
--container-architecture linux/amd64 \
--secret-file .secrets \
--bind \
--job deploy

```

---
키 파일의 값 한줄의 문자열로 변환
```shell
KEY_FILE_PATH="/Users/yonghwankim/.ssh/gcp_vm"
awk '{printf "%s\\n", $0}' "$KEY_FILE_PATH" | sed 's/\\n$//'
```

