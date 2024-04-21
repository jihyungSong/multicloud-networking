# Azure NAT Gateway 구성 실습

외부 통신이 단절된 Private Subnet 에서 아웃 바운드로 인터넷 통신이 가능하도록 구성해주는 NAT Gateway 를 사용하도록 설정합니다.  
아래 순서로 진행됩니다.  

1. Private Subnet 에 Virtual Machine 배포
2. Virtual Machine 에 Bastion 을 통해 접속
3. 인터넷 통신 불가 확인
4. NAT Gateway 구성
5. Private Subnet 에 NAT Gateway 설정
6. Private Route Table 경로 수정
7. 아웃 바운드 인터넷 통신 확인

---
## 1. Private Subnet 에 Virtual Machine 배포
Private 서브넷에 새로운 Virtual Machine 을 추가로 구성합니다.  
상단 검색에서 `가상 머신` 로 검색 후, `+ 만들기 ` -> `Azure 가상 머신` 을 클릭하여 생성 작업을 수행합니다.  
  
[기본 사항]
1. 리소스 그룹: `{skuserNN}-resource-group`
2. 가상 머신 이름: `{skuserNN}-private-vm`
3. 지역: Virtual Network 와 동일 지역 선택
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
1. 가상 네트워크: `{skuserNN}-azure-vnet` 선택
2. 서브넷: `{skuserNN}-azure-subnet-02` Private Subnet 선택
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
  
모든 검토 완료후 만들기를 클릭하여 Virtual Machine 을 배포 합니다.  
  

## 2. Virtual Machine 에 Bastion 을 통해 접속
Virtual Machine 배포가 완료되었다면, Bastion 을 통해 접속 합니다.  
상단 검색에서 `가상 머신` 을 선택 후, 이전 단계에서 배포한 `{skuserNN}-private-vm` 를 클릭 합니다.  
서브 메뉴에서 `베스천` 을 클릭 후, 아래와 같이 선택 후 연결을 시도 합니다.  

- 인증 유형: `VM 암호`
- 사용자 이름: `ubuntu`
- VM 암호: VM 생성시 설정한 암호 입력
- 새 브라우저 탭에서 열기: 체크

## 3. 인터넷 통신 불가 확인
터미널 접속 상태에서 아래와 같이 패키지 레포지토리 업데이트를 위한 외부 통신 요청을 수행해 봅니다.

```
ubuntu@skuserNN-private-vm:~# sudo apt update
...
Err:1 http://azure.archive.ubuntu.com/ubuntu jammy InRelease
  Could not connect to azure.archive.ubuntu.com:80 (20.196.152.132), connection timed out
Err:2 http://azure.archive.ubuntu.com/ubuntu jammy-updates InRelease
  Unable to connect to azure.archive.ubuntu.com:http:
Err:3 http://azure.archive.ubuntu.com/ubuntu jammy-backports InRelease
  Unable to connect to azure.archive.ubuntu.com:http:
Err:4 http://azure.archive.ubuntu.com/ubuntu jammy-security InRelease
  Unable to connect to azure.archive.ubuntu.com:http:
...
```

인터넷 통신이 막힌 상태이기 때문에 패키지 레포 업데이트가 실패하는 것이 확인 됩니다.  

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
상단 검색에서 `가상 네트워크` 로 검색 후, `{skuserNN}-azure-vnet` 를 선택합니다. 
서브 메뉴 중 `서브넷` 을 선택 후, Private Subnet 으로 설정된 `{skuserNN}-azure-subnet-02` 를 선택 합니다. 
NAT Gateway 항목에 위에서 생성한 `{skuserNN}-nat-gateway` 를 선택 후 저장 합니다. 


## 6. Private Route Table 경로 수정
상단 검색에서 `경로 테이블` 로 검색 후, `{skuserNN}-private-route-table` 를 선택합니다. 
서브 메뉴의 `경로` 선택 후, `InternalOnly` 경로를 삭제 합니다.  

## 7. 아웃 바운드 인터넷 통신 확인
터미널 접속 상태에서 위와 동일한 명령어를 실행하여 외부 통신이 성공하는지 확인합니다.  

```
ubuntu@skuserNN-private-vm:~# sudo apt update
...
Hit:1 http://azure.archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://azure.archive.ubuntu.com/ubuntu jammy-updates InRelease [119 kB]
Get:3 http://azure.archive.ubuntu.com/ubuntu jammy-backports InRelease [109 kB]
..
Fetched 31.0 MB in 6s (5458 kB/s)                          
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
30 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

(추가) NAT Gateway 의 모니터링 메트릭을 통해 Packet 통신이 NAT Gateway 를 통해서 수행되고 있음을 확인 합니다.  

---

NAT Gateway 구성 실습이 완료되었습니다.
