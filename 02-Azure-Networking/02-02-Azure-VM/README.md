# Azure Virtual Network - VM 배포 실습

위 단계에서 구성한 Virtual Network 에 Virtual Machine 을 배포하여 통신 확인을 수행합니다.
아래 구성은 [Azure Virtual Network 기본 환경 구성](../02-01-Azure-VNET/README.md) 에 이어서 진행합니다.  
아래 순서로 진행됩니다.  
  
1. Public 서브넷에 Virtual Machine 배포
2. Network Security Group 의 규칙 수정
3. 접속 확인  

---
## 1. Public 서브넷에 Virtual Machine 배포
상단 검색에서 `가상 머신` 로 검색 후, `+ 만들기 ` -> `Azure 가상 머신` 을 클릭하여 생성 작업을 수행합니다.  
  
[기본 사항]
1. 리소스 그룹: `{skuserNN}-resource-group`
2. 가상 머신 이름: `{skuserNN}-web-instance`
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
2. 서브넷: `{skuserNN}-azure-subnet-01` 퍼블릭 서브넷 선택
3. 공용 IP: `{skuserNN}-web-instance-ip` 새로 만드는 중 선택
4. NIC 네트워크 기본 보안: `고급` 선택 (새로 만드는 중)
5. VM 삭제시 공용 IP 및 NIC 삭제: 체크
6. 부하 분산 옵션: `없음` 선택

[관리]
기본 선택된 것 그대로 사용합니다.  

[모니터링]
기본 선택된 것 그대로 사용합니다.  

[고급]
사용자 지정 데이터에 아래 스크립트를 복사 붙여 넣기 합니다.  

```
#!/bin/bash

sudo apt update -y
sudo apt install -y nginx
sudo apt install -y jq
sudo systemctl start nginx
sudo systemctl enable nginx

IP_ADDR=$(curl -s -H Metadata:true --noproxy "*" "http://169.254.169.254/metadata/instance?api-version=2021-02-01" | jq '.network.interface[].ipv4.ipAddress[].privateIpAddress')

sudo rm -rf /var/www/html/*
sudo chown -R ubuntu:ubuntu /var/www
sudo chmod 2775 /var/www

sudo echo "<h1>Azure</h1> <p>SERVER IP : $IP_ADDR</p>" >> /var/www/html/index.html
sudo chown -R root:root /var/www
```


## 2. Network Security Group 의 규칙 수정
위 Virtual Machine 은 배포와 함께 웹서버로 자동 구성 됩니다.  
웹 서버의 정상 동작하기 위해서는 Network Security Group 에서 80 포트를 오픈해 주어야 합니다.  

상단 검색에서 `네트워크 보안 그룹` 으로 검색 후, Virtual Machine 배포와 함께 생성했던 `{skuserNN}-web-instance-nsg` 을 선택 합니다.
서브 메뉴에서 `인바운드 보안 규칙` 을 선택하고 `+ 추가`를 클릭하여 신규 규칙을 추가합니다. 

- 소스: `Any`
- 원본 포트 범위: `*`
- 대상 주소: `Any`
- 서비스: `HTTP`
- 대상 포트 범위: `80`
- 프로토콜: `TCP`
- 작업: `허용`
- 우선 순위: `100`
- 이름: `AllowAnyHTTPInbound`

위와 같이 설정 후 인바인드 규칙을 추가 합니다.  


## 3. 접속 확인
최종 배포 완료된 Virtual Machine 에서 웹 서비스가 정상 동작 중인지 확인 합니다.  
상단 검색에서 `가상 머신` 로 검색 후, 위에서 생성한 가상 머신 `{skuserNN}-web-instance` 을 선택 합니다. 
서버의 공용 IP 주소 정보를 복사 하여, 웹 브라우저를 통해 접속을 시도합니다.  
  
정상 응답이 오는지 최종 확인 합니다.  

---

Virtual Machine 배포 실습이 완료 되었습니다.
