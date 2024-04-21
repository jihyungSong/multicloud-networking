# AWS VPC Peering 구성

위 단계에서 구성한 VPC 와 Private 통신을 하기 위한 VPC 를 추가 구성하여, 통신을 확인 합니다. 
아래 구성은 [AWS VPC 기본 환경 구성 (3)](../01-03-AWS-VPC-EC2/README.md)에 이어서 진행합니다. 
아래 순서로 진행됩니다.

1. VPC 추가 생성
2. Subnet 생성
3. Peering Conneciton 생성
4. Route Table 설정
5. EC2 인스턴스 생성
6. Security Group Rule 설정
7. EC2 인스턴스 통신 확인


---
## 1. VPC 추가 생성
지금까지 생성한 VPC 와 Peering 으로 연결하기 위한 추가 VPC 를 생성합니다.  
VPC 메뉴로 이동 하여, VPC 생성을 시작 합니다.  

- VPC 이름 : `{skuserNN}-vpc-peering` (ex. `skuser01-vpc-peering`)
- IPv4 CIDR : `10.20.0.0/16`
- Tenancy : `Default`

## 2. Subnet 구성  
VPC 내부를 구성하는 Subnet 을 생성합니다.  
VPC 의 `Subnets` 메뉴로 이동하여, Subnet 생성을 시작합니다.  

- VPC ID: 이전 단계에서 생성한 VPC 를 선택 합니다.  
- Subnet 은 총 2개를 생성 합니다.  
    * **Subnet 1**
      * 이름 : `{skuserNN}-vpc-peering-subnet-01` (ex. `skuser01-vpc-peering-subnet-01`)
      * Availability Zone : 해당 리전의 `첫번째` AZ 를 선택합니다.
      * IPv4 CIDR: `10.20.1.0/24`
    * **Subnet 2**
      * 이름 : `{skuserNN}-vpc-peering-subnet-02` (ex. `skuser01-vpc-peering-subnet-02`)
      * Availability Zone : 해당 리전의 `두번째` AZ 를 선택합니다. 
      * IPv4 CIDR: `10.20.2.0/24`

## 3. Peering Connection 생성
두 VPC 를 Peering 으로 연결합니다.  
VPC 메뉴 중 피어링 연결로 이동하여, 피어링 연결 생성을 선택합니다.  

- 이름: `{skuserNN}-vpc-peering`
- VPC ID(요청자): `{skuserNN}-aws-vpc`
- 피어링할 다른 VPC 선택: `내계정` 선택 후, 리전은 `현재 리전` 선택
- VPC ID (수락자): `{skuserNN}-vpc-peering`
- 태그:
  * 키: `Name`  값: `{skuserNN}-vpc-peering`

모든 설정을 마친 후, 피어링 생성을 클릭 합니다.  
피어링 상태를 보면, `수락 대기중` 인 것을 확인 합니다.  
현재 요청자와 수락자가 같은 계정 내 동일 리전이므로, 해당 피어링 선택 후 `작업`에서 `요청 수락` 버튼을 클릭 합니다.  
요청 수락이 완료 되면 피어링의 최종 상태가 `활성`으로 변경 된 것이 확이 됩니다. 


## 4. Route Table 설정
Peering Connection 으로 VPC 를 연결하였지만, 실제 통신이 되기 위해서는 Route 구성이 필요합니다.  
Route 구성은 양쪽 VPC 에서 모두 설정해 주어야 합니다.  
  
VPC 의 `Route tables` 메뉴로 이동 합니다. 
Route Table 중 `{skuserNN}-vpc-peering` 의 VPC ID 로 검색하여 설정을 추가할 Route Table 을 선택 합니다. 

  *- 참고 : 해당 Route table 는 자동으로 생성되었기 때문에 Name 은 비어 있으며, Main : Yes 로 구성되어 있습니다. Name 을 편집하여 `skuserNN-vpc-peering-route-table` 로 이름을 수정합니다.* 


해당 Route table 을 선택하고, 하단 서브탭에서 `Routes` 을 클릭 후, `Edit routes` 로 추가 경로를 설정합니다.
* Destination : `10.10.0.0/16`
* Target : 피어링 연결 선택 후, 위에서 생성한 Peering Connection (`{skuserNN}-vpc-peering`) 을 선택 

이번에는, VPC `{skuserNN}-aws-vpc` 에 적용되어 있는  VPC ID 로 검색하여 설정을 추가할 Route Table 을 선택 합니다. 
VPC Peering 통신시, 다른 VPC 에 있는 `{skuserNN}-route-table` 와 `{skuserNN}-private-route-table` 에 각각 Route 설정을 추가 합니다.  

* Destination : `10.20.0.0/16`
* Target : Peering Connection (`{skuserNN}-vpc-peering`) 을 선택 


## 5. EC2 인스턴스 생성
VPC Peering 통신 확인을 위한 EC2 인스턴스를 생성합니다.   
EC2 의 인스턴스 메뉴로 이동하여, '인스턴스 시작' 을 클릭합니다.  

- 이름 : `{skuserNN}-peering-instance`
- OS 이미지: Ubuntu 22.04 LTS (HVM) SSD Volume Type
- 아키텍쳐: 64비트(x86)
- 인스턴스 유형: t3.micro
- 키페어: 위에서 생성한 키페어 선택 (`{skuserNN}-{리전코드}`)
- VPC: 이전 단계에서 생성한 `{skuserNN}-vpc-peering` 선택
- 서브넷: `{skuserNN}-vpc-peering-subnet-01` 또는 `{skuserNN}-vpc-peering-subnet-02` 선택
- 퍼블릭 IP: `비활성화`
- 방화벽(보안그룹): 기존 보안 그룹 선택 - `default`


## 6. Security Group Rule 설정
마지막으로, 서로 다른 VPC 에 존재하는 EC2 인스턴스 사이 통신을 확인하기 위해 각각의 인스턴스에 적용되어 있는 Security Group 의 룰을 추가로 설정하여 줍니다.  
먼저, `{skuserNN}-vpc-peering` VPC 에 배포되어 있는 EC2 인스턴스에 적용된 `default` 보안 그룹을 선택하여 아래와 같이 인바운드 규칙을 추가 합니다.  

- 인바운드 규칙에서 `규칙 추가`  
  * 유형: `모든 트래픽`
  * 프로토콜: `전체`
  * 포트범위 : `전체`
  * 소스 : `사용자 지정` `10.10.0.0/16`

그 다음으로, 이전 단계에서 생성한 `{skuserNN}-aws-vpc` VPC 에 생성되어 있는 `{skuserNN}-admin-sg` 와 `{skuserNN}-private-sg` 에 각각 인바운드 규칙을 추가합니다.  

- 인바운드 규칙에서 `규칙 추가`  
  * 유형: `모든 트래픽`
  * 프로토콜: `전체`
  * 포트범위 : `전체`
  * 소스 : `사용자 지정` `10.20.0.0/16`


## 7. EC2 인스턴스 통신 확인
각 VPC 에 배포된 EC2 인스턴스 간 통신을 확인 합니다.  

### (추가)  
Private DNS 로 통신 할 수 있도록 확인합니다.  
양쪽 VPC 의 속성 중 `DNS 호스트 이름 활성화` 를 체크하여 활성화 합니다.  


### (추가1)
Public DNS 로 요청시 Public IP 가 아닌, Private IP 로 응답이 오도록 구성합니다.  
VPC Peering 의 DNS 설정 편집을 통해 요청자/수락자 DNS 확인을 활성화 합니다.  
---

AWS VPC Peering 구성 실습이 완료 되었습니다.
