---
layout: post
title:  "금주의 AWS 실패사례 - Stack 구축"
date:   2017-07-07 12:00:00 +0900
categories: AWS Failure
comments: true
---
# 이번 주 AWS를 사용하면서 겪은 실패 사례

## 진행 작업
- 다른 팀의 프로젝트용 환경 구축
  - 환경 Stack
    - VPC > Private/Public Subnet 2AZ
    - NAT Gateway 1, Bastion Host 1
    - Multi-AZ RDS(MySQL)
    - S3 > CloudFront
    - Elastic Beanstalk
    - S3, RDS migration작업
  - 부득이 하게 수작업으로 하나하나 등록함, 이때 겪은 케이스들

- 발생 문제
  1. CloudFront로 S3접근하기 안됨
    - S3 접근 시 CloudFront에서만 가능하도록 restriction설정 -> S3에 policy에 Origin Access Identity설정 해야됨됨
  2. RDS Multi-AZ로 등록하기 안됨
    - Subnet Group에서 분명 Multi-AZ로 등록했지만 안됨 -> AZ f에서는 RDS가 등록이 안됨(신규 AZ문제 인듯?)
    - RDS를 넣을 Private Subnet하나를 AZ c로 변경
  3. Elastic Beanstalk 등록 시 ELB등록 문제 발생
    - 위 RDS Multi-AZ문제와 연관
    - Public / Private Subnet이 같은 AZ가 아니면 Public Subnet에 ELB 등록시 EC2 Instance가 있는 Private Subnet과 AZ가 달라 오류 발생
    - EC2 Instance를 넣을 Public Subnet도 AZ c로 변경
  4. Public Subnet AZ f에 있던 Bastion Host 이동
    - Subnet삭제 시 해당 Subnet에 들어있는 모든 Instance와 ENI 삭제해야 함
    - Bastion Host삭제하고 NAT Gateway삭제 -> Public Subnet새로 생성 후 그 안에 재생성
  5. RDS migration 에서 오류
    - DMS를 통해 DB 복제
    - 순서 EndPoint등록, Subnet Group생성, Replica 생성, Task 등록 및 실행
    - Task등록 시 %(모든) scheme과 table복제 설정 -> mysql scheme(아마 system table)에서 에러발생
    - mysql scheme은 exclude처리 -> Task complete됨(실질적인 문제는 없음)
    - Task등록 시 log설정을 해두면 CloudWatch에서 migration log 조회 가능
  6. S3 migration 후 권한 문제
    - s3 sync 명령으로 account간 migration진행
    - 이때 S3 Bucket policy에서 허용하지 않은 계정 accessKey로 진행 : 왜 오류가 안났는지는 의문
    - target account에 없는 owner로 migration됨 -> 조회 및 role변경 권한이 없어서 데이터 사용 불가
    - 결국 s3 cp 명령으로 덮어쓰기
    - 참고로 9GB/Hour 정도 속도
 - 향후 생각나면 추가 예정
