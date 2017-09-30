---
layout: post
title:  "금주의 AWS 실패사례 - Data Migration"
date:   2017-07-11 12:00:00 +0900
categories: AWS Failure Migration
comments: true
---
# 이번 주 AWS를 사용하면서 겪은 실패 사례

## 데이터 백업 및 마이그레이션
- 운영중인 데이터의 백업과 마이그레이션
  - 대상
    - MySQL RDS
    - MongoDB Instances
  - 이론만 배웠지 실제로 해보지 않으면 겪지 않을 이슈들
  - MongoDB backup의 절차
    1. MongoDB의 IP확인 -> EC2 Instance 찾아 VPC 확인 -> VPC내에 cluster확인
    2. 작업중인 primary instance를 피해 slave instance의 정보 확인
    3. 해당 instance와 같은 subnet에 백업용 EC2 Instance 생성 + EBS attach + Security Group설정
    4. 백업용 instance에 ssh로 접속하여 백업 shell script 생성
    5. crontab에 해당 shell script 등록

- 발생 문제
  1. MySQL migration
    - 생각해본 가용 방법들
      - MySQL의 volume(storage)에 대해 Snapshot 생성 -> S3에 업로드/다운로드 -> 새 DB에 Snapshot 복원
      - MySQL Workbench를 이용해 export -> 새 DB에서 import
      - Amazon Data Migration Service(DMS) 사용
    - 가장 간편하고 Amazon에서 지원하기 때문에 RDS랑 호환 잘 될 것으로 생각
      - DMS의 문제점 : View나 Function등 Table외 요소는 복사 안됨, 데이터 타입 안맞음
      - 결국 Workbench로 export / import 수행
  2. MySQL backup
    - 생각해본 가용 방법들
      - Replica 생성 후 연결 끊기 - 향후 복원시 Replica를 Master로 바꾸면 안될까?
      - MySQL Workbench로 export
    - 가장 정확하고 간편할 것 같아 Workbench로용 export 수행
  3. MongoDB backup
    - 생각해본 가용 방법들
      - MongoDB Instance의 volume을 Snapshot 처리
      - Replica 생성 후 연결 끊기
      - mongodump 사용
    - 실물을 확인 할 수 있는 mongodump 사용
      - mongodump의 버전이 충분히 높아야 됨. --version으로 확인 후 3.0 이하라면 update필요
      - Local PC - Remote DB 간 속도가 느려 엄청 오래 걸림(25Gb에 약 8시간 소요)
      - 결국 MongoDB Instamce와 동일한 VPC에 instance를 추가하여 여기서 mongodump 수행(3Gb에 약 5분 소요)
    - crontab을 이용해 주기적으로 backup작업 수행
      - crontab은 EC2 Instance의 시간을 쓰기 때문에 schedule설정 시 주의
      - mongodump의 --out이 mount된 volume인 것 확인 필수
      - 불필요한 collectdion은 --excludeCollection을 활용

- MongoDB backup 하면서 배운 지식들
  1. mongodump : MongoDB 데이터를 bson형식으로 백업. table이나 collection단위로 선택가능하고 압축도 가능
  2. crontab : Linux에서 스케쥴을 등록하여 작업 수행가능. crontab 실행 path가 home보다 parent인점 참고
  3. EC2 Instance에 volume 붙였다 떼기 : 신규 volume등록 시 이를 포맷(mkfs) 후 마운트(mount) 시켜 사용해야 함
  4. 여러 Linux 명령어들 : df -h, chmod +x, lsblk, grep, date +%T 등

- 기타
  - shell script를 이용해 주기적으로 MongoDB를 dump하여 S3에 업로드하는 cron job도 생성 가능

개삽질의 연속..
