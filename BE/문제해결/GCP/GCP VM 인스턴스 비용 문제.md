
### 배경
- VM 인스턴스 1대를 운용하고 1달동안 사용량 및 비용 측정 표입니다.
- 현재는 구글 프리티어를 사용하고 있지만 프리티어 기간이 지나면 비용이 발생합니다.
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112121735.png)

## 원인
- 디스크 타입으로 Balanced PD 타입(pd-balanced)을 사용해서 상대적으로 비용이 발생함

### 해결 방법
- 디스크 타입을 pd-standard로 변경함


---

## pd-balanaced에서 pd-standard로 디스크 타입 변경
### 스냅샷 생성
![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112124417.png)

![](BE/문제해결/GCP/refImg/Pasted%20image%2020251112124431.png)

