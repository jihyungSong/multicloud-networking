# AWS VPC 

위 단계에서 구성한 VPC 에 EC2 인스턴스를 배포하여 통신 확인을 수행합니다.
아래 구성은 [AWS VPC 기본 환경 구성 (1)](../01-01-AWS-VPC/README.md)과 [AWS VPC 기본 환경 구성 (2)](../01-02-AWS-VPC-NAT-Gateway/README.md)에 이어서 진행합니다. 
아래 순서로 진행됩니다.

1. Keypair 생성
2. Security Group 생성
3. EC2 인스턴스 배포
4. EC2 인스턴스 통신 확인


---
## 1. Private Subnet 구성
VPC 의 `Subnets` 메뉴로 이동하여, Subnet 을 추가로 생성 합니다.

- VPC ID: 이전 단계에서 생성한 VPC (`{skuserNN}-aws-{리전코드}-vpc`) 를 선택 합니다.
- Subnet 은 총 2개를 추가로 생성 합니다.
    * **Subnet 3**
      * 이름 : `{skuserNN}-aws-{리전코드}-private-subnet-03` (ex. `skuser01-aws-us-east-1-private-subnet-03`)
      * Availability Zone : 해당 리전의 `첫번째` AZ 를 선택합니다. 해당 예제는 us-east-1 리전에 구성 하였으므로, `us-east-1a` 를 선택 합니다.
      * IPv4 CIDR: `10.10.3.0/24`
    * **Subnet 4**
      * 이름 : `{skuserNN}-aws-{리전코드}-private-subnet-04` (ex. `skuser01-aws-us-east-1-private-subnet-04`)
      * Availability Zone : 해당 리전의 `두번째` AZ 를 선택합니다. 해당 예제는 us-east-1 리전에 구성 하였으므로, `us-east-1b` 를 선택 합니다.
      * IPv4 CIDR: `10.10.4.0/24`

## 2. Private Route Table 구성
위에서 추가 생성한 Subnet 은 Internet 통신을 허락하지 않도록 구성하기 위해, 추가로 Route Table 을 생성합니다. 
VPC 의 `Route tables` 메뉴로 이동 하여, `라우팅 테이블 생성`을 시작합니다.  

- 이름 : `skuserNN-aws-리전코드-private-route-table`
- VPC : `skuserNN-aws-리전코드-vpc` VPC 의 ID 

Route Table 생성시, 생성 완료된 Route Table 메뉴로 자동 이동합니다.  
하단 탭에서 `서브넷 연결` 을 선택하여 해당 Route Table 에 위에서 만든 Private 용 서브넷을 연결합니다.  


## 3. NAT Gateway 구성
Private Subnet 의 아웃 바운드에 대해 인터넷 통신을 허용하기 위해 NAT Gateway 를 구성 합니다.
VPC 의 `NAT gateways` 메뉴로 이동하여, 생성을 시작합니다.

- Name: `{skuserNN}-aws-{리전코드}-vpc-natgw`
- Subnet : 외부 통신이 가능한 Public Subnet 을 지정합니다. `skuserNN-aws-리전코드-subnet-01` 를 선택합니다.
- 연결 유형: 퍼블릭
- 탄력적 IP 할당 ID : 새로운 EIP 할당을 위해 `탄력적 IP 할당` 버튼을 클릭합니다. 이때 신규 할당 ID 가 자동 선택되는지 확인 합니다. 

NAT gateway 는 생성 완료까지 5분 내외 시간이 필요합니다.


## 4. NAT Gateway 를 위한 Route Table 경로 업데이트
위 1번 단계에서 생성한 Private Subnet 은 외부 통신이 완전히 차단되어 있는 Subnet 입니다.  
아웃 바운드로 외부 통신을 허용하기 위해 3번 단계에서 생성한 NAT Gateway 로 트래픽이 전달될 수 있도록 Route Table  경로를 추가 합니다.  

VPC 의 `Route tables` 메뉴로 이동 하여, 위에서 생성한 Private Route Table(`skuserNN-aws-리전코드-private-route-table`) 을 선택 합니다.  
하단의 라우팅 탭을 선택하여, `라우팅 편집 버튼`을 클릭하여 수정합니다.  

- `라우팅 추가` 버튼 클릭  
- 대상: `0.0.0.0/0` 을 위에서 생성한 NAT Gateway 선택 후 저장   

---

AWS VPC 구성 업데이트가 완료 되었습니다.
