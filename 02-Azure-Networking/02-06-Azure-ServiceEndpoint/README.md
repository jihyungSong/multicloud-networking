# Azure Virtual Network - Service Endpoint 실습

Azure 서비스에 대한 요청을 인터넷이 아닌 Service Endpoint 를 통해 통신되도록 설정합니다. 
이번 실습에서는 Azure 서비스 중 Storage Account 를 대상으로 수행하게 됩니다.  
아래 순서로 진행됩니다. 

1. Storage Account 생성


---
## 1. Storage Account 생성
상단 검색에서 `스토리지 계정` 으로 검색 후, `+ 만들기 ` 를 클릭하여 생성 작업을 수행합니다.  
  
[기본 사항]
1. 리소스 그룹: `{skuserNN}-resource-group`
2. 스토리지 계정 이름: `{skuserNN}storage`
3. 지역: Virtual Network 와 동일 지역 선택
4. 성능: `표준` 선택
5. 중복: `LRS(로컬 중복 스토리지)` 선택

  
[고급]
1. REST API 작업을 위한 보안 전송 필요: 체크
2. 개별 컨테이너에 대한 익명 액세스 허용: 체크 해제
3. 스토리지 계정 키 액세스 사용: 체크
4. Azure Portal에서 Microsoft Entra 인증 기본값 사용: 체크 해제
5. 최소 TLS 버전: `버전 1.2` 선택
6. 복사 작업에 대해 허용된 범위(미리 보기): `모든 스토리지 계정에서` 선택

나머지 설정은 기본 설정 그대로 사용합니다. 
  
[네트워킹]
1. 네트워크 액세스: `선택한 가상 네트워크 및 IP 주소에서 퍼블릭 액세스 사용` 선택
2. 가상 네트워크 구독: 현재 사용 중인 구독 선택
3. 가상 네트워크: `{skuserNN}-azure-vnet` 선택
4. 서브넷: `{skuserNN}-azure-subnet-02` Private Subnet 선택
5. 라우팅 기본 설정: `Microsoft 네트워크 라우팅` 선택

[데이터 보호]
기본 선택된 것 그대로 사용합니다.  

[암호화]
기본 선택된 것 그대로 사용합니다.  

모든 검토 완료후 만들기를 클릭하여 Storage Account 를 생성 합니다.  
  


---

Virtual Network - Service Endpoint 구성 실습이 완료되었습니다.
