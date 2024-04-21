# Azure NAT Gateway 구성 실습

외부 통신이 단절된 Private Subnet 에서 아웃 바운드로 인터넷 통신이 가능하도록 구성해주는 NAT Gateway 를 사용하도록 설정합니다.  
아래 순서로 진행됩니다.  

1. Private Subnet 에 Virtual Machine 배포
2. Virtual Machine 에 Bastion 을 통해 접속
3. 인터넷 통신 불가 확인
4. NAT Gateway 구성
5. Private Subnet 에 NAT Gateway 설정
6. 아웃 바운드 인터넷 통신 확인

---
## 1. Private Subnet 에 Virtual Machine 배포

## 2. Virtual Machine 에 Bastion 을 통해 접속

## 3. 인터넷 통신 불가 확인

## 4. NAT Gateway 구성
상단 검색에서 `NAT 게이트웨이` 로 검색 후, `+ 만들기`를 클릭하여 신규 NAT Gateway 를 생성합니다.  
  
- 리소스 그룹: `{skuserNN}-resource-group`
- 이름: `{skuserNN}-nat-gateway`
- 지역: Virtual Network 와 동일 지역 선택
- 가용성 영역: `구역 없음` 선택
- TCP 유휴 시간 제한(분): 4분 (default)

[아웃바운드 IP]
- 공용 IP: `새 공용 IP 주소 만들기` 클릭
 * 이름: `{skuserNN}-nat-ip` 설정 후 확인.

 [서브넷]
 - 가상 네트워크: `{skuserNN}-azure-vnet`
 - 서브넷: Public Subnet `{skuserNN}-azure-subnet-01` 선택

위와 같이 설정 후 NAT Gateway 를 생성합니다. 
  

## 5. Private Subnet 에 NAT Gateway 설정
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

## 6. 아웃 바운드 인터넷 통신 확인

---

NAT Gateway 구성 실습이 완료되었습니다.
