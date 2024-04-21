# Azure Virtual Network Peering 실습

위 단계에서 구성한 Virtual Network 와 Private 통신을 하기 위한 Virtual Network 를 추가 구성하여, 통신을 확인 합니다. 
아래 순서로 진행됩니다.

1. Virtual Network 추가 생성
2. Peering Conneciton 생성
3. Virtual Machine 생성
4. Virtual Machine 통신 확인
  

---
## 1. Virtual Network 추가 생성
상단 검색에서 `가상 네트워크` 로 검색 후, `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.  

[기본구성]
- 리소스 그룹: `{skuserNN}-resource-group` 선택
- VPC 이름 : `{skuserNN}-vnet-peering` (ex. `skuser01-vnet-peering`)
- 지역: 적절한 지역 선택 후 생성 합니다.

[보안]
추가 구성 없이 넘어 가도록 합니다.  

[IP주소]
- IP 주소 공간: `172.18.0.0/16`
- Subnet 1 개를 생성 합니다.
    * **Subnet**
      * 서브넷 용도: Default
      * 이름: `{skuserNN}-peering-subnet-01` (ex. `skuser01-peering-subnet-01`)
      * `IP 주소 공간 포함` 체크
      * 시작 주소: `172.18.1.0`
      * 크기: `/24(주소 256개)`
      * 나머지 설정은 변경 없음

[태그]
별도 태그 설정 없이 넘어 가도록 합니다. 
  
최종 `만들기` 를 눌러 Virtual Network 를 생성하도록 합니다. 


## 2. Peering Conneciton 생성
위에서 생성한 `{skuserNN}-vnet-peering` 선택 후, 서브 메뉴에서 `피어링` 선택. `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.

- 이 가상 네트워크의 피어링 링크 이름: `{skuserNN}-vnet-peering`
 * 'skuserNN-vnet-peering'에서 피어링된 가상 네트워크에 액세스 허용: 체크
 * 'skuserNN-vnet-peering'에서 피어링된 가상 네트워크이(가) 전달한 트래픽 수신 허용: 체크
 * 'skuserNN-vnet-peering' 게이트웨이 또는 경로 서버가 트래픽을 피어링된 가상 네트워크에게 전달하도록 허용: 체크
 * 'skuserNN-vnet-peering'이(가) 피어링된 가상 네트워크의 원격 게이트웨이 또는 경로 서버를 사용하도록 설정 : `체크 해제`
- 원격 가상 네트워크의 피어링 링크 이름: `{skuserNN}-vnet-peering`
- 가상 네트워크 배포 모델: `리소스 관리자` 선택
- 구독: 현재 구독 선택
- 가상 네트워크: 연결 대상 Virtual Network `{skuserNN}-azure-vnet` 선택
 * 'skuserNN-azure-vnet'에서 피어링된 가상 네트워크에 액세스 허용: 체크
 * 'skuserNN-azure-vnet'에서 피어링된 가상 네트워크이(가) 전달한 트래픽 수신 허용: 체크
 * 'skuserNN-azure-vnet' 게이트웨이 또는 경로 서버가 트래픽을 피어링된 가상 네트워크에게 전달하도록 허용: 체크
 * 'skuserNN-azure-vnet' 이(가) 피어링된 가상 네트워크의 원격 게이트웨이 또는 경로 서버를 사용하도록 설정 : `체크 해제`

위와 같이 선택 후 추가 버튼을 눌러 Peering 을 생성 합니다. 


## 3. Virtual Machine 생성
피어링 연결 테스트를 위해 `{skuserNN}-vnet-peering` Virtual Network 에 VM 을 생성합니다.  
상단 검색에서 `가상 머신` 로 검색 후, `+ 만들기 ` -> `Azure 가상 머신` 을 클릭하여 생성 작업을 수행합니다.  
  
[기본 사항]
1. 리소스 그룹: `{skuserNN}-resource-group`
2. 가상 머신 이름: `{skuserNN}-peer-vm`
3. 지역: `{skuserNN}-vnet-peering` Virtual Network 와 동일 지역 선택
4. 가용성 옵션: `인프라 중복이 필요하지 않습니다` 선택
5. 보안 유형: `표준` 선택
6. 이미지: `Ubuntu 22.04 LTS - x64 Gen2` 선택
7. VM 아키텍쳐: `x64` 선택
8. Azure Spot 할인으로 실행: 체크 해제
9. 크기: 모든 크기 보기 -> `B1ls` 선택 (1vCPU 0.5 RAM(GiB))
10. 인증 형식: `암호` 선택 후, 사용자 이름 (`ubuntu`) 와 암호 설정
11. 공용 인바운드 포트: `없음` 선택
  
[디스크]
1. OS 디스크 크기: `이미지 기본값(30GiB)` 선택
2. OS 디스크 유형: `표준 SSD(로컬 중복 스토리지)` 선택
3. VM 으로 삭제: 체크
4. 키 관리: 플랫폼 관리형 키
5. Ultra DisK 호환성 사용: 체크 해제
  
[네트워킹]
1. 가상 네트워크: `{skuserNN}-vnet-peering` 선택
2. 서브넷: `{skuserNN}-peering-subnet-01` 서브넷 선택
3. 공용 IP: `없음` 선택
4. NIC 네트워크 기본 보안: `없음` 선택
5. VM 삭제시 NIC 삭제: 체크
6. 부하 분산 옵션: `없음` 선택

[관리]
기본 선택된 것 그대로 사용합니다.  

[모니터링]
기본 선택된 것 그대로 사용합니다.  

[고급]
기본 선택된 것 그대로 사용합니다.  

모든 설정 완료 후, 만들기를 클릭하여 가상 머신을 생성 합니다. 

## 4. Virtual Machine 통신 확인
배포된 가상 머신을 선택하고, Peering 을 통해 다른 Virtual Network 에 배포된 VM 사이에 Private IP 로 통신이 되는지 확인 합니다. 


---

VNet Peering 구성 실습이 완료되었습니다.
