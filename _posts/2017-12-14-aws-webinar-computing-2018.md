---
layout: post
title:  "AWS 컴퓨팅 분야 신규 서비스 소개 - 2018년"
date:   2017-12-14 11:00:00 +0900
categories: AWS EC2
comments: true
---
# AWS 컴퓨팅 서비스 특징 및 최신 트렌드
해당 포스팅은 2017년 12월 14일에 있었던 "AWS re:Invent 특집(1) - 컴퓨팅 분야 신규 서비스 소개"를 간단히 정리한 것 입니다.

## AWS 컴퓨팅 서비스 특징
  * Primitives : 레고 블럭처럼 조립 가능
  * Fully-Managed : 관리형 서비스, 관리 부담/비용을 제거
  * Serverless : 미래지향적, 개발자에게 서버 관리/비용의 부담을 줄임
  * Inter-Region : 지역에 관계없이 최고의 성능을 낼 수 있는 환경 제공

## 컴퓨팅 서비스 : Instance, Containers, Serverless
* Instance : CPU - C type, RAM - R type, GPU - G type
  + 기존에는 Hypervisor아래에 물리영역이 동일했지만, Nitro system(hypervisor)을 도입하여 Network(2013)와 Storage(2014), Management/Security/Monitoring(2016)을 물리적으로 분리
    - 참고 : [[AWS re:invent 2017: Evolution of the EC2 Host](https://www.awsgeek.com/posts/reinvent2017-evolution-of-the-ec2-host/)]
  + 높은 BandWidth의 네트워크로 EC2인스턴스와 Storage, 관리서비스를 연동
  + 이렇게 분리함으로써 VM Ware같은 타사 솔루션을 AWS에 탑재 가능해짐 - VM AWS
  + 신규 EC2 Bare Metal 인스턴스 출시 - 가상화 할 수 없는, 물리적인 라이선스 제약 등에 활용
  + 신규 EC2 type
    - T2 Unlimited : 이를 활성화 하면 1일치 Credit을 추가 부여함
    - M5.24XLARGE : 고성능 M4대비 14% 성능향상
    - H1.16XLARGE : 빅데이터에 최적화(D2보다 CPU와 BandWidth가 높음)

* Containers : ECS, ECR, Kubenetes
  + AWS내 EC2의 63%의 점유율 kubenetes -> 관리형 kubenetes 출시 (EKS, 2018 출시 예정)
  + ECS의 컨테이너만 쓰고 싶다 -> AWS Fargate
    - 서버리스 컨테이너 서비스 (ECS내 위치, EKS 출시때 같이 출시), 컨테이너만 띄우고 서버 클러스터는 알아서 설정

* Serverless : 서버관리X, 유연한확장, 고가용성, 사용한 만큼만 지불 - 개발자는 비즈니스 로직에 집중
  + AWS Lambda
    - Event driven, S3나 SNS등 event source로 부터 trigger, IoT등으로 확대
    - Lambda 콘솔 개선, API Gateway VPC 연동, 메모리 확장(->3Gb), 동시 제어 지원, 추가 언어 지원
  + Serverless App Repository
    - 람다 함수를 배포/구매 eco system(기존에는 BluePrint가 있었지만 배포가 가능)

## 추가 정보
* 기타
  + AWS Cloud9
    - Cloud 기반 IDE - 빠른 배포, 강력한 AWS 연동, 페어 프로그래밍, 다양한 디버깅 도구 -> CodeStar로 바로 배포!
  + AWS UserGroup의 관심있는 서비스
    - Aurora Serverless, EKS, AWS Cloud9, Fargate 등..
  + AWS의 신규 서비스 개발 순서
    - API -> SDK/CLI -> Console (콘솔은 API의 일부만 구현, 잘 쓰기 위해서는 API를 쓰세요)
