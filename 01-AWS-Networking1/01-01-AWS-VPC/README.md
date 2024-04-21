# AWS VPC 기본 환경 구성 (1)

AWS VPC 환경을 구성 합니다.  
아래 순서로 진행됩니다.

1. VPC 생성
2. Subnet 구성
3. Internet Gateway 구성
4. Route Table 구성

---
## 1. VPC 생성
AWS Management Console 로그인 한 후, 적당한 Region 을 선택 합니다.  
해당 실습의 경우, N.Virginia 리전 (us-east-1) 를 선택 하여 구성 하였습니다. (어떤 리전이든 상관 없습니다)  

VPC 메뉴로 이동 하여, VPC 생성을 시작 합니다.
- VPC 이름 : `{skuserNN}-aws-vpc` (ex. `skuser01-aws-vpc`)
- IPv4 CIDR : `10.10.0.0/16`
- Tenancy : `Default`


## 2. Subnet 구성
VPC 내부를 구성하는 Subnet 을 생성합니다.  
VPC 의 `Subnets` 메뉴로 이동하여, Subnet 생성을 시작합니다.

- VPC ID: 이전 단계에서 생성한 VPC 를 선택 합니다.
- Subnet 은 총 2개를 생성 합니다.
    * **Subnet 1**
      * 이름 : `{skuserNN}-aws-subnet-01` (ex. `skuser01-aws-subnet-01`)
      * Availability Zone : 해당 리전의 `첫번째` AZ 를 선택합니다. 해당 예제는 us-east-1 리전에 구성 하였으므로, `us-east-1a` 를 선택 합니다.
      * IPv4 CIDR: `10.10.1.0/24`
    * **Subnet 2**
      * 이름 : `{skuserNN}-aws-subnet-02` (ex. `skuser01-aws-subnet-02`)
      * Availability Zone : 해당 리전의 `두번째` AZ 를 선택합니다. 해당 예제는 us-east-1 리전에 구성 하였으므로, `us-east-1b` 를 선택 합니다.
      * IPv4 CIDR: `10.10.2.0/24`

## 3. Internet Gateway 구성
VPC 의 인터넷 통신을 위해 Internet Gateway 를 구성 합니다.
VPC 의 `Internet gateways` 메뉴로 이동하여, Internet gateway 생성을 시작합니다.

- Name: `{skuserNN}-aws-vpc-igw`

Internet gateway 는 생성후, 인터넷 통신이 필요한 VPC 에 attach 하는 작업이 필요 합니다.  
생성한 Internet gateway 를 선택 후, `Actions` -> `Attach to VPC` 를 선택 합니다.  
VPC 는 위에서 생성한 `skuserNN-aws-vpc` VPC 를 선택 합니다.  

attach 작업이 완료 되면, Internet gateway 항목에서 VPC ID 가 맵핑된 내용을 확인 가능 합니다.


## 4. Route Table 구성
Internet gateway 를 VPC 와 연동하였지만, 실제 Subnet 에 인터넷 통신 되려면, Subnet 에서 발생되는 트래픽이 Internet gateway 로 통신 되도록 Route 구성이 필요합니다.  
VPC 의 `Route tables` 메뉴로 이동 하여, 경로 설정을 구성합니다.  
Route tables 리스트에서 해당 VPC ID (`skuserNN-aws-vpc` VPC 의 ID) 로 검색 하여, VPC 가 생성 될때 자동으로 구성된 기본 Route table 을 선택합니다.  

  *- 참고 : 해당 Route table 는 자동으로 생성되었기 때문에 Name 은 비어 있으며, Main : Yes 로 구성되어 있습니다. Name 을 편집하여 `skuserNN-route-table` 로 이름을 수정합니다.* 


해당 Route table 을 선택하고, 하단 서브탭에서 `Routes` 을 클릭 후, `Edit routes` 로 추가 경로를 설정합니다.
* Destination : `0.0.0.0/0`
* Target : 이전 단계에서 생성한 Internet Gateway 선택

---

AWS VPC 구성이 완료 되었습니다.
