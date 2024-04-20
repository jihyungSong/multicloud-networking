# AWS VPC 기본 환경 구성 (3) - EC2 인스턴스 배포

위 단계에서 구성한 VPC 에 EC2 인스턴스를 배포하여 통신 확인을 수행합니다.
아래 구성은 [AWS VPC 기본 환경 구성 (1)](../01-01-AWS-VPC/README.md)과 [AWS VPC 기본 환경 구성 (2)](../01-02-AWS-VPC-NAT-Gateway/README.md)에 이어서 진행합니다. 
아래 순서로 진행됩니다.

1. Keypair 생성
2. Security Group 생성
3. EC2 인스턴스 배포
4. EC2 인스턴스 통신 확인


---
## 1. Keypair 생성
적절한 리전을 선택 후, EC2 의 `Keypair` 메뉴로 이동하여, 키페어를 생성 합니다.

- 이름: `{skuserNN}-{리전코드}`
- 키페어 유형: RSA
- 프라이빗 키 파일 형식: 윈도우에서 PuTTY 로 접속하는 경우 `.ppk` 를 선택. 다른 터미널 프로그램을 사용한다면 `.pem` 을 선택


## 2. Security Group 생성
EC2 에 적용할 보안 그룹을 생성 합니다.  
EC2 의 `Security Group` 메뉴로 이동 하여, `보안 그룹 생성` 을 선택합니다.

- 보안 그룹 이름 : `{skuserNN}-{리전코드}-web-sg`
- 설명: `For Web server` 
- VPC: 이전 단계에서 생성한 VPC 를 선택
- 인바운드 규칙에서 `규칙 추가`  
  * 유형: `HTTP`
  * 프로토콜: `TCP`
  * 포트범위 : `80`
  * 소스 : `Anywhere-IPv4` (0.0.0.0/0)
- 태그:
  * 키: `Name`  값: `{skuserNN}-{리전코드}-web-sg`

보안 그룹 생성 완료 후, 추가로 보안 그룹을 생성합니다.  

- 보안 그룹 이름 : `{skuserNN}-admin-sg`
- 설명: `For Admin Access` 
- VPC: 이전 단계에서 생성한 VPC 를 선택
- 인바운드 규칙에서 `규칙 추가`  
  * 유형: `SSH`
  * 프로토콜: `TCP`
  * 포트범위 : `22`
  * 소스 : `내 IP`
- 태그:
  * 키: `Name`  값: `{skuserNN}-admin-sg`

- 보안 그룹 이름 : `{skuserNN}-private-sg`
- 설명: `For Private Instance` 
- VPC: 이전 단계에서 생성한 VPC 를 선택
- 인바운드 규칙에서 `규칙 추가`  
  * 유형: `모든 트래픽`
  * 프로토콜: `전체`
  * 포트범위 : `전체`
  * 소스 : `사용자 지정` `10.10.0.0/16`
- 태그:
  * 키: `Name`  값: `{skuserNN}-private-sg`



## 3. EC2 인스턴스 배포
Public Subnet 과 Private Subnet 에 각각 EC2 인스턴스를 생성합니다.   
EC2 의 인스턴스 메뉴로 이동하여, '인스턴스 시작' 을 클릭합니다.  

- 이름 : `{skuserNN}-private-instance`
- OS 이미지: Ubuntu 22.04 LTS (HVM) SSD Volume Type
- 아키텍쳐: 64비트(x86)
- 인스턴스 유형: t3.micro
- 키페어: 위에서 생성한 키페어 선택 (`{skuserNN}-{리전코드}`)
- VPC: 이전 단계에서 생성한 `{skuserNN}-aws-{리전코드}-vpc` 선택
- 서브넷: 프라이빗 서브넷 `{skuserNN}-aws-{리전코드}-private-subnet-03` 또는 `{skuserNN}-aws-{리전코드}-private-subnet-04` 선택
- 퍼블릭 IP: `비활성화`
- 방화벽(보안그룹): 기존 보안 그룹 선택 - `{skuserNN}-private-sg`

나머지 설정은 그대로 적용 후 인스턴스 시작을 클릭 합니다.  
추가로 인스턴스를 하나 더 생성합니다.  

- 이름 : `{skuserNN}-web-instance`
- OS 이미지: Ubuntu 22.04 LTS (HVM) SSD Volume Type
- 아키텍쳐: 64비트(x86)
- 인스턴스 유형: `t3.micro`
- 키페어: 위에서 생성한 키페어 선택 (`{skuserNN}-{리전코드}`)
- VPC: 이전 단계에서 생성한 `{skuserNN}-aws-{리전코드}-vpc` 선택
- 서브넷: 프라이빗 서브넷 `{skuserNN}-aws-{리전코드}-subnet-01` 또는 `{skuserNN}-aws-{리전코드}-subnet-02` 선택
- 퍼블릭 IP: `활성화`
- 방화벽(보안그룹): 기존 보안 그룹 선택 - `{skuserNN}-web-sg`
- 고급 세부 정보 - 사용자 데이터에 아래와 같이 입력
"""
#!/bin/bash

sudo apt update -y
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx

IP_ADDR=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)

sudo rm -rf /var/www/html/*
sudo chown -R ubuntu:ubuntu /var/www
sudo chmod 2775 /var/www

sudo echo "<h1>HELLO</h1> <p>SERVER IP : $IP_ADDR</p>" >> /var/www/html/index.html
sudo chown -R root:root /var/www
"""

설정을 완료 후 인스턴스 시작을 클릭해 Web 서버용 인스턴스를 배포합니다.  


## 4. EC2 인스턴스 통신 확인


---

AWS EC2 인스턴스 실습을 완료하였습니다. 
