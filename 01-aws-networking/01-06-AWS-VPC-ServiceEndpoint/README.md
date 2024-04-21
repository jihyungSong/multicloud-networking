# AWS VPC - Service Endpoint (1)

AWS 서비스에 대한 요청을 Internet Gateway 가 아닌 Private Endpoint 를 통해 통신되도록 설정합니다. 
이번 실습에서는 AWS 서비스 중 S3 를 대상으로 수행하게 됩니다.  
아래 순서로 진행됩니다. 

1. EC2 인스턴스에 IAM Role 부여
2. EC2 인스턴스에 AWS CLI 구성
3. VPC Endpoint 용 Security Group 생성
4. S3 용 VPC Endpoint 생성
5. EC2 인스턴스에서 S3 통신 확인


---
## 1. EC2 인스턴스에 IAM Role 부여
인스턴스에서 S3 로 API 요청 테스트를 하려면, 먼저 AWS CLI 를 설치해야 합니다.  
Private 서브넷에 배포된 인스턴스(`{skuserNN}-private-instance`) 에 IAM 역할 Role 을 설정합니다.  
  
* EC2 인스턴스에 IAM 역할을 부여하려면, 적절한 IAM 역할을 먼저 생성해야 합니다. 해당 예제에서는 미리 생성된 `S3FullAccessRole` 이라는 이름의 역할을 대상으로 수행합니다. 만약 적절한 역할이 없다면 먼저 IAM 역할 부터 생성하도록 합니다.    
  
인스턴스 선택 후, `작업` -> `보안` -> `IAM 역할 수정` 을 선택합니다.  
IAM 역할로 `S3FullAccessRole` 을 선택 합니다.  
  

## 2. EC2 인스턴스에 AWS CLI 구성
먼저, 인스턴스(`{skuserNN}-private-instance`)에 접속 합니다.  
이전 단계에서 생성한 EC2 Connect Endpoint 를 통해 접근하도록 합니다.  
  
Linux 명령어로 패키지 업데이트를 수행합니다.
```
ubuntu@ip-10-10-xx-xx:~$ sudo apt update
```
  
패키지 업데이트가 완료되었으면, AWS CLI 패키지를 설치 합니다.  
```
ubuntu@ip-10-10-xx-xx:~$ sudo apt install awscli -y
```
  
AWS CLI 패키지 설치가 왼료 되면, AWS CLI 명령어가 정상 동작되는지 테스트 해 봅니다. 
```
ubuntu@ip-10-10-xx-xx:~$ aws sts get-caller-identity
{
    "UserId": "AROAZZDIVYV2XXXXXX:i-0exxxxxxx",
    "Account": "67238NNNNNNN",
    "Arn": "arn:aws:sts::67238NNNNNNN:assumed-role/S3FullAccessRole/i-0exxxxxxx"
}
```

* 위 명령어는 AWS CLI 사용시 어떤 계정이 어떠한 역할(Role) 로 요청이 수행되는지 확인 하는 명령입니다.  
  
아래는 현재 계정에 있는 S3 버킷 리스트를 조회하는 명령어 입니다.    
```
ubuntu@ip-10-10-xx-xx:~$ aws s3 ls
```

## 3. VPC Endpoint 용 Security Group 생성
VPC Endpoint 를 생성하기 전에 해당 Endpoint 로 통신될 ENI 에 적용될 Security Group 을 먼저 생성합니다.  
  
EC2 의 `Security Group` 메뉴로 이동 하여, `보안 그룹 생성` 을 선택합니다.

- 보안 그룹 이름 : `{skuserNN}-s3-endpoint-sg`
- 설명: `For S3 VPC Endpoint` 
- VPC: `{skuserNN}-aws-vpc`
- 태그:
  * 키: `Name`  값: `{skuserNN}-s3-endpoint-sg`

## 4. S3 용 VPC Endpoint 생성
S3 와의 통신을 수행할 VPC Endpoint 를 VPC 에 생성합니다.  

- 이름 태그: `{skuserNN}-s3-endpoint`
- 서비스 범주: `AWS 서비스` 선택
- 서비스: `s3` 로 검색 하여, `com.amazonaws.{리전}.s3` 서비스 이름 선택. 유형으로는 `Interface` 선택.
- VPC: `{skuserNN}-aws-vpc`
- 서브넷: 가용 영역 모두 체크. 서브넷 ID 는 각 가용 영역 중 Private 서브넷으로 선택. 
- 보안 그룹: 위에서 생성한 `{skuserNN}-s3-endpoint-sg` 선택.
- 정책: `전체 액세스` 
- 태그:
  * 키: `Name`  값: `{skuserNN}-s3-endpoint`

모든 설정 완료 후 엔드포인트 생성을 수행합니다.  

## 5. EC2 인스턴스에서 Endpoint 를 통한 S3 통신 확인


---

AWS VPC Endpoint 실습을 완료하였습니다. 