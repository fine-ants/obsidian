
## 배경
2024-12월에 대한 청구서를 확인하였는데 예상치 못한 5.43$ 과금이 발생하였습니다.
![[Pasted image 20250102140910.png]]

## 원인
- 12월에 로드 밸런서 ALB 추가하게 되었습니다. 그러나 ALB 추가시 public subnet을 최소 2개를 요구하게 됩니다.
- 기존 프리티어 사용시에는 public subnet을 1개만 사용하였고 public IPv4에 대해서 750시간을 지원하여 과금이 발생하지 않았지만 추가적인 public subnet 발생으로 인해서 추가적인 IPv4을 사용하게 되었습니다.

## 해결 방법
- public subnet 2개를 IPv4를 할당하지 않고 IPv6으로 할당하도록 변경합니다.
- ALB을 IP 주소 유형을 퍼블릭 IPv4가 없는 듀얼 스택으로 다시 생성합니다.


### VPC 새로운 IPv6 CIDR 추가
1. VPC 서비스의 VPC 페이지로 이동합니다.
![[Pasted image 20250102141905.png]]

2. IPv6 CIDR를 추가할 VPC를 선택한 다음에 작업->CIDR 편집 버튼을 클릭합니다.
![[Pasted image 20250102141939.png]]

3. CIDR 편집 페이지에서 새 IPv6 CIDR 추가 버튼을 클릭하여 새로운 IPv6 CIDR을 추가합니다. 추가한 다음에 닫기 버튼을 클릭합니다.
![[Pasted image 20250102143441.png]]
- VPC의 CIDR 블록이 xxx:xxx:xxx:xxx::/56인 경우에는 128

![[Pasted image 20250102142115.png]]


### Public Subnet IPv6 할당
1. VPC 서비스의 서브넷 페이지로 이동합니다.
![[Pasted image 20250102141627.png]]

2. Public Subnet 한개를 선택한 다음에 작업->IPv6 CIDR 편집 버튼을 클릭하여 이동합니다.
![[Pasted image 20250102141734.png]]

3. 