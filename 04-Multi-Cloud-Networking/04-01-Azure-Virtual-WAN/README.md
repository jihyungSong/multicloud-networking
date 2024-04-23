# Azure Virtual WAN 환경 구성

1. Virtual WAN 생성
2. Hubs 생성과 함께 VPN Gateway 배포

---

## 1. Virtual WAN 생성
상단 검색에서 `가상 WAN` 으로 검색 후, `+ 만들기 `를 클릭하여 생성 작업을 수행합니다.  

* 리소스 그룹 : `{skuserNN}-resource-group`
* 지역 : 적절한 지역 선택
* 이름 : `{skuserNN}-azure-vwan`
* 유형 : `표준` 선택

## 2. 가상 Hubs 생성과 함께 VPN Gateway 배포

* 지역 : Virtual WAN 과 동일한 지역 선택
* 이름 : `{skuserNN}-azure-hub`
* 허브 프라이빗 주소 공간 : `192.168.0.0/24`
* 가상 허브 용량 : `2 Routing Infrastructure Units, 3 Gbps Router, Supports 2000 VMs`
* 허브 라우팅 기본 설장 : `VPN`

[사이트 대 사이트]
* Site to site(VPN Gateway)를 만드시겠습니까? : `Yes`
  * AS Number : `65515`
  * 게이트웨이 배율 단위 : `1 scale unit - 500 Mbps x 2` 선택
  * 라우팅 기본 설정: `Microsoft 네트워크` 선택

나머지(`지점 대 사이트`, `ExpressRoute`)는 생성하지 않고 가상 Hub 생성을 시작합니다.  

---
Azure Virtual WAN 환경 구성 실습이 완료되었습니다. 