# AWS VPC - EC2 Connect Endpoint 

EC2 인스턴스에 대해 외부 22번(SSH) 오픈 없이 Endpoint 를 통해 연결 할 수 있도록 설정합니다.  
아래 순서로 진행됩니다.  

1. Endpoint 생성
2. Security Group 규칙 수정
2. EC2 연결 확인



---
## 1. Endpoint 생성
VPC 의 `Subnets` 메뉴로 이동하여, Subnet 을 추가로 생성 합니다.

- 이름 태그: `{skuserNN-ec2-endpoint}`
- 서비스 범주 : `EC2 인스턴스 연결 엔드포인트` 선택
- VPC: `{skuserNN}-aws-vpc` 선택
- 보안 그룹 선택 X
- 서브넷: `{skuserNN}-aws-private-subnet-04` 선택


## 2. Security Group 규칙 수정
Endpoint 를 사용하게 되면, 외부로 부터 SSH 오픈 없이 설정된 Subnet 에 배포된 ENI 를 통해 SSH 통신이 수행됩니다.  
해당 ENI 에서 EC2 인스턴스로 접근할 수 있도록 Security Group 룰을 수정합니다.  

Public Subnet 에 배치되어 있는 인스턴스에 적용된 Security Group 중 `{skuserNN}-admin-sg` 를 선택하고, 인바운드 규칙을 편집 합니다.

- 기존 SSH 외부 IP 로 노출되어 있던 인바운드 규칙을 삭제 합니다.
- 인바운드 규칙에서 `규칙 추가`  
  * 유형: `SSH`
  * 프로토콜: `TCP`
  * 포트범위 : `22`
  * 소스 : `사용자 지정` `10.10.0.0/16`

위와 같이 수정 후 저장 합니다.  


## 3. EC2 연결 확인
Endpoint 를 통해 EC2 인스턴스 연결을 확인 합니다.  
`{skuserNN}-web-instance` 와 `{skuserNN}-private-instance` 를 대상으로 Endpoint 를 통해 연결을 시도해 봅니다.  
먼저, `{skuserNN}-web-instance` 를 선택 후, `연결`을 클릭 합니다.  
  
- EC2 인스턴스 연결 탭 선택
- 연결 유형: `EC2 인스턴스 연결 엔드포인트를 사용하여 연결` 선택
- EC2 인스턴스 연결 엔드포인트로 위에서 생성한 엔드포인트 선택
- 사용자 이름: `ubuntu`

연결을 시도합니다. 브라우저에 새로운 탭이 열리고 터미널이 정상 연결됨을 확인 합니다. 

### (추가) 클라이언트 IP 보존 설정된 EC2 Connect Endpoint 추가


---

EC2 Connect Endpoint 구성 실습이 완료 되었습니다.
