
## 배경
Spring 서버를 대상으로 힙 덤프 파일을 생성한 다음에 메모리 누수 리포트를 생성하였습니다. 리포트 개요는 다음과 같습니다.
![[Pasted image 20250422131610.png]]

Problem Suspect 1이 어느 부분에서 메모리 누수가 발생하는지 분석합니다. Problem Suspect 1의 설명은 다음과 같습니다.
![[Pasted image 20250422131709.png]]
![[Pasted image 20250422131720.png]]

Details 버튼을 클릭하여 상세 정보를 확인합니다. Detail 창에서 Biggest Instances를 보면 점유하고 있는 메모리 대부분이 AspectExpressionPointcut 클래스가 차지하고 있는 것을 볼수 있습니다. 맨 위를 하나 선택해서 어느 Aspect에서 발생하는지 확인해봅니다.
![[Pasted image 20250422132011.png]]

클래스 중 하나를 분석하기 위해서 Path To GC Roots -> with all references를 선택합니다.
![[Pasted image 20250422132136.png]]

실행 결과를 보면 중간에 WatchListService가 보입니다. 
![[Pasted image 20250422132234.png]]

## 원인
## 해결 방법