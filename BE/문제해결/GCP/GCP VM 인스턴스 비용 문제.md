
### 배경
- VM 인스턴스 1대를 운용하고 1달동안 사용량 및 비용 측정 표입니다.
- 현재는 구글 프리티어를 사용하고 있지만 프리티어 기간이 지나면 비용이 발생합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112121735.png)

## 원인
- 디스크 타입으로 Balanced PD 타입(pd-balanced)을 사용해서 상대적으로 비용이 발생함
- 디스크의 스냅샷 스케줄러에 의해서 비용이 발생함

### 해결 방법
- 디스크 타입을 pd-standard로 변경함
- 스냅샷 스케줄러를 삭제함



---

## pd-balanaced에서 pd-standard로 디스크 타입 변경
### 스냅샷 생성
현재 실해중인 vm 인스턴스를 대상으로 스냅샷을 생성합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112124417.png)

![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112124431.png)

스냅샷 생성 확인
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112124908.png)

### 디스크 만들기
스냅샷 메뉴에서 "디스크 만들기"를 선택하여 디스크를 생성합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112124930.png)

디스크 생성 정보를 입력합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112125043.png)

디스크 유형에서 **"표준 영구 디스크"**를 선택합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112125102.png)

디스크 생성 확인
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112125304.png)

### 부팅 디스크 교체
VM 인스턴스 중지
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112125901.png)

인스턴스 중지 확인
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112130041.png)

현재 인스턴스의 디스크 확인
- 실행 결과를 보면 pd-balanaced 디스크를 사용중인 것을 볼수 있음
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112130108.png)

기존 부팅 디스크 분리
- 인스턴스 상세 화면에서 수정화면으로 들어가고 부팅 디스크 섹션에서 "부팅 디스크 분리"를 선택합니다.
- 부팅 디스크 분리전에 삭제 규칙에서 인스턴스 삭제시 디스크 유지를 선택합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112130741.png)

기존 디스크 연결
- 부팅 디스크 구성을 선택합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112130849.png)

기존 디스크 탭에서 이전에 생성한 pd-standard 디스크를 선택합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112131028.png)

부팅 디스크 선택 확인
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112131052.png)

VM 인스턴스 재시작- 
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112131307.png)

부팅 디스크 변경 확인
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112131430.png)

---

## 스냅샷 스케줄러 삭제
스냅샷 메뉴로 이동합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112131657.png)




## References
- https://docs.cloud.google.com/compute/docs/disks/performance?hl=ko
- https://docs.cloud.google.com/compute/docs/disks/detach-reattach-boot-disk?hl=ko