# Azure Bastion 구성 실습

Virtual Machine에 대해 외부 22/3389 (SSH/RDP) 오픈 없이 Bastion 을 통해 연결 할 수 있도록 설정합니다.  
아래 순서로 진행됩니다.  
  
1. Bastion 용 Subnet 구성
2. Bastion 구성
3. Bastion 을 통한 Virtual Machine 연결

---
## 1. Bastion 용 Subnet 구성
Bastion 을 구성하려면, Basion 을 배포할 Virtual Network 에 `AzureBastionSubnet` 이라는 이름의 `/26` 이상의 서브넷이 존재해야 합니다.  
상단 검색에서 `가상 네트워크` 로 검색 후, 이전 단계에서 생성한 Virtual Network `{skuserNN}-azure-vnet` 를 선택 합니다.  
  
서브 메뉴에서 `서브넷` 을 선택하고, `+ 서브넷`을 클릭하여 해당 Virtual Network 에 신규 서브넷을 생성합니다. 

- 이름: `AzureBastionSubnet`
- 서브넷 주소: `172.16.100.0/24`

위와 같이 설정 후 서브넷을 생성합니다. 
  

## 2. Bastion 구성
상단 검색에서 `Bastion` 로 검색 후, `+ 만들기 ` 로 생성 작업을 수행합니다.  

- 리소스 그룹: `{skuserNN}-resource-group`
- 이름: `{skuserNN}-bastion`
- 지역: Virtual Network 와 동일 지역 선택
- 계층: `기본` 선택 (인스턴스 수 2개 자동 선택)
- 가상 네트워크: `{skuserNN}-azure-vnet` 선택
- 서브넷: `AzureBastionSubnet(172.16.100.0/24)` 선택
- 공용 IP 주소: `새로 만들기` 선택
- 공용 IP 주소 이름: `{skuserNN}-vnet-ip`

위와 같이 설정후 Bastion 을 생성합니다.  


## 3. Bastion 을 통한 Virtual Machine 연결
Bastion 을 통해 Virtual Machine 에 접속을 테스트 해봅니다.  
상단 검색에서 `가상 머신` 을 선택 후, 이전 단계에서 배포한 `{skuserNN}-web-instance` 를 클릭 합니다.  
서브 메뉴에서 `베스천` 을 클릭 후, 아래와 같이 선택 후 연결을 시도 합니다.  

- 인증 유형: `VM 암호`
- 사용자 이름: `ubuntu`
- VM 암호: VM 생성시 설정한 암호 입력
- 새 브라우저 탭에서 열기: 체크

브라우저의 새 탭에서 터미널 접속이 성공하는 것을 확인 합니다.  


---

Bastion 구성 실습이 완료되었습니다.
