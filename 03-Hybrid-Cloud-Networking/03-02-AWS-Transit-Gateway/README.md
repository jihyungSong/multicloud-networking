# On-premise 와 AWS 연동 - Transit Gateway VPC 연동 실습

1. Transit Gateway 구성
2. Transit Gateway 와 AWS VPC 연동

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

---

Transit Gateway 와 VPC 연동 실습을 완료하였습니다.
