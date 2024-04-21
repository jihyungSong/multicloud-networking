# Azure Virtual Network 기본 환경 구성

Azure Virtual Network 환경을 구성 합니다.  
아래 순서로 진행됩니다.

1. Resource Group 생성
2. Virtual Network 생성 및 Subnet 구성
3. Route Table 구성
4. Network Security Group 생성
5. Private Subnet 에 NSG 연결

---
## 1. Resource Group 생성
실습을 위해 Resource Group 을 생성합니다.  
상단 검색에서 `리소스 그룹` 으로 검색 후, `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.  

- 리소스 그룹 이름: `{skuserNN}-resource-group` (ex. `skuser01-resource-group`)
- 영역: 적절한 영역 선택 후 생성 합니다.


## 2. Virtual Network 생성 및 Subnet 구성
상단 검색에서 `가상 네트워크` 로 검색 후, `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.  

[기본구성]
- 리소스 그룹: `{skuserNN}-resource-group` 선택
- VPC 이름 : `{skuserNN}-azure-vnet` (ex. `skuser01-azure-vnet`)
- 지역: 적절한 지역 선택 후 생성 합니다.


[보안]
추가 구성 없이 넘어 가도록 합니다.  

[IP주소]
- IP 주소 공간: `172.16.0.0/16`
- Subnet 은 총 2개를 생성 합니다.
    * **Subnet 1**
      * 서브넷 용도: Default
      * 이름: `{skuserNN}-azure-subnet-01` (ex. `skuser01-azure-subnet-01`)
      * `IP 주소 공간 포함` 체크
      * 시작 주소: `172.16.1.0`
      * 크기: `/24(주소 256개)`
      * 나머지 설정은 변경 없음
    * **Subnet 2**
      * 이름 : `{skuserNN}-azure-subnet-02` (ex. `skuser01-azure-subnet-02`)
      * `IP 주소 공간 포함` 체크
      * 시작 주소: `172.16.2.0`
      * 크기: `/24(주소 256개)`

[태그]
별도 태그 설정 없이 넘어 가도록 합니다. 
  
최종 `만들기` 를 눌러 Virtual Network 를 생성하도록 합니다. 
  


## 3. Route Table 구성
상단 검색에서 `경로 테이블` 로 검색 후, `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.  

- 리소스 그룹: `{skuserNN}-resource-group` (ex. `skuser01-resource-group`)
- 지역: Virtual Network 와 동일 지역 선택
- 이름: `{skuserNN}-private-route-table`
- 게이트웨이 경로 전파: `Yes` 선택

위와 같이 설정 하고 경로 테이블을 생성합니다.  
공유기 생성 완료 후 `경로` 서브 메뉴로 이동하여 `+ 추가` 를 수행하여 경로를 추가합니다.  
  
- 경로 이름: `InternalOnly`
- 대상 유형: `IP 주소` 선택
- 대상 IP주소/CIDR 범위: `0.0.0.0/0`
- 다음 홉 형식: `없음`

위와 같이 설정하고 경로를 추가 합니다.  
Route Table 경로 설정이 완료되었다면, Subnet 에 해당 경로를 적용 합니다.  
가상 네트워크 메뉴로 이동하여, 위에서 생성한 Virtual Network `{skuserNN}-azure-vnet` 를 선택합니다.  
서브 메뉴의 서브넷을 선택 하고 Private Subnet 으로 동작할 `{skuserNN}-azure-subnet-02` 를 선택합니다.  
  
`{skuserNN}-azure-subnet-02` 의 경로 테이블을 `{skuserNN}-private-route-table` 로 지정 후 저장 합니다.  


## 4. Network Security Group 생성
상단 검색에서 `네트워크 보안 그룹` 으로 검색 후, `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.  

- 리소스 그룹: `{skuserNN}-resource-group` 
- 이름: `{skuserNN}-private-subnet-nsg`
- 지역: Virtual Network 와 동일 지역으로 선택


## 5. Private Subnet 에 NSG 연결
위에서 만든 Network Security Group 을 Private Subnet 에 적용해 보도록 합니다.  
가상 네트워크 `{skuserNN}-azure-vnet` 를 선택 후, 서브 메뉴 중 `서브넷` 을 선택하여, Private Subnet 인 `{skuserNN}-azure-subnet-02` 를 선택 합니다. 

- 네트워크 보안 그룹: `{skuserNN}-private-subnet-nsg` 

설정 완료 후 저장 합니다.  

---

Azure Virtual Network 구성이 완료 되었습니다.
