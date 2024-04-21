# On-premise 와 AWS 연동 - Transit Gateway 실습

1. Transit Gateway 구성
2. Transit Gateway 와 AWS VPC 연동
3. On-premise 연결을 위한 Customer Gateway 생성
4. On-premise 연결을 위한 VPN Connection 구성

---

## 1. Transit Gateway 구성

먼저, AWS VPC 와 On-premise 사이를 Relay 역할을 담당하는 Transit Gateway 를 구성합니다.  
AWS VPC 가 자리잡고 있는 리전으로 이동하여, VPC 페이지에서 `Transit Gateway` 메뉴로 이동하여 `Create transit gateway` 를 수행 합니다.  

- name : `{skuserNN}-transit-gateway`
- ASN : 비워두면 자동으로 `64512` 로 구성
- DNS 지원: 체크
- VPN ECMP 지원: 체크
- 기본 라우팅 테이블 연결: 체크
- 기본 라우팅 테이블 전파: 체크
- 멀티캐스트 지원: 체크 해제
- 공유 첨부 파일 자동 수락: 체크 해제



## 2. Transit Gateway 와 AWS VPC 연동

Transit gateway 구성이 완료되었으면, AWS VPC 와 Transit gateway 를 연결하도록 합니다.  
`Transit gateway attachments` 메뉴로 이동하여, `Create transit gateway attachment` 를 수행 합니다.  

- name : `{skuserNN}-tgw-attach`
- Transit gateway ID : 이전 단계에서 생성한 `{skuserNN}-transit-gateway` 선택
- 연결유형(Attachment type) : `VPC`
- VPC 연결
 * DNS 지원: 체크
 * IPv6 지원: 체크 해제
 * 어플라이언스 모드 지원: 체크 해제
- VPC ID : `{skuserNN}-aws-vpc` 선택
 * Subnet ID 모든 Availability Zone 선택

구성 완료 후, 시간이 지나면, State 가 `Available` 로 변경되는 것을 확인 합니다.  

이후, AWS VPC(`{skuserNN}-aws-vpc`) 로 이동하여, On-premise network 와 네트워크 통신이 가능하도록, 경로 설정을 추가하도록 합니다.  
`Route tables` 메뉴로 이동하여, `{skuserNN}-aws-vpc` 에 연결된 Route table 인 `skuserNN-route-table` 와 `skuserNN-private-route-table` 선택 하고, 하단 `Route` 탭으로 이동하여, `Edit routes` 를 수행합니다.

- Destination : `192.168.0.0/16` (On-premise 네트워크의 CIDR)
- Target : Transit gateway 선택 (`{skuserNN}-tgw-attach`)

## 3. On-premise 연결을 위한 Customer Gateway 생성

On-premise 의 VPN Gateway 역할을 할 Customer Gateway 를 생성합니다.  
해당 Gateway 는 실제 VPN 장비를 생성하는 것은 아닙니다. 이는 이후 5번 단계에서 실제 VPN Gateway 를 배포 하며, Customer Gateway 는 이에 대한 준비 단계로 이해하시면 됩니다.  

**(사전 준비)**  
설정을 시작하기 전에, Customer Gateway 에서 사용할 EIP 를 할당 받도록 합니다.  
EIP 는 `Elastic IPs` 메뉴로 이동하여 `Allocate Elastic IP address` 를 수행 합니다. (`{skuser30}-cgw-eip`)  
*(참고) 이때, 해당 EIP 의 `Allocation ID`(`eipalloc` 으로 시작하는 ID) 를 기억해 두어야 합니다. 이후 설정에 필요 합니다.*

EIP 할당이 완료 되었다면, VPC 페이지에서 `Customer Gateway` 메뉴로 이동 하고, `Create customer gateway` 를 수행 합니다. 

* name : `{skuserNN}-on-premise-cgw`
* BGP ASN : `65000`
* IP Address : 할당 받은 EIP 주소
* Tags : `Name: {skuserNN}-on-premise-cgw`


## 4. On-premise 연결을 위한 VPN Connection 구성

On-premise 구간을 연동할 VPN Connection 설정을 시작 합니다.
`Site-to-Site VPN connections` 메뉴로 이동하여, `Create VPN connection` 을 수행 합니다.  

- 이름 태그(Name) : `{skuserNN}-on-prem-conn`
- 대상 게이트웨이 유형(Target gateway type) : `Transit gateway`
- Transit gateway : 이전 단계에서 생성한 `{skuserNN}-transit-gateway` 선택
- 고객 게이트웨이(Customer gateway) : `기존 Existing`
- 고객 게이트웨이 ID(Customer gateway ID) : `{skuserNN}-on-premise-cgw`
- 라우팅 옵션(Routing options) : `동적 Dynamic (requires BGP)`
- 터널 내부 IP 버전(Tunnel inside IP version) : `IPv4`
- Tags : `Name : {skuserNN}-on-prem-conn`

---
