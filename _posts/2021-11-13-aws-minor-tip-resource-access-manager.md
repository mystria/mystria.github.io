---
layout: post
title:  "AWS 미립자 팁 - Resource Access Manager"
date:   2021-11-13 20:00:00 +0900
categories: AWS RAM VPC
comments: true
---

# Amazon Resource Access Manager, RAM
AWS에는 별의별 기능이 다 있는데, 이번에는 RAM, Resource Access Manager이다.  
이름 그대로 AWS의 자원을 서로 다른 계정들이 접근하도록 하는, 공유하는 방법이다.  
보통 AWS의 best practice는 각 팀끼리 계정을 고립(isolated)시켜 사용하길 권장한다. 하지만 가끔 중복 자원 방지(= 비용 절감) 등을 위해 자원을 함께 사용하고 싶을 수도 있을 것이다.

## Resource Sharing
RAM은 AWS의 다양한 자원(주로 네트워크와 정책 등)을 다른 계정과 공유(Share)하는 용도이다.  
공유(Shares) > 공유한 자원(Shared resources) 계층으로, 한 '공유'아래에 여러 '자원'을 묶어서 공유할 수 있다.  
내가 공유한 것(Shared by me)와 내가 공유 받은 것(Shared with me)으로 구분되며, 공유 받은 것들은 해당 자원의 console에서 자연스럽게 나의 자원처럼 표시된다.  
비용과 보안 문제가 있을 수 있으므로 여러 제약사항이 있는데, 리소스 별로 공유할 수 있는 대상(Principals)이 다르다. 공유 제한을 위해 Settings에서 Organization 공유 기능을 활성화할 수 있다.

* 참조
  + [AWS RAM 가이드 문서](https://docs.aws.amazon.com/ram/latest/userguide/getting-started-sharing.html)
  + [AWS 블로그, RAM 소개](https://aws.amazon.com/ko/blogs/korea/new-aws-resource-access-manager-cross-account-resource-sharing/)

공유 자원의 종류에 따라 다양한 사용법이 있겠지만, 본인은 Subnet만 공유하여 사용해 보았다.

## Shared VPCs(Shared Subnets)
만약 A 계정에서 Subnet(&VPC)을 B 계정에게 공유해 주면, B 계정의 사용자들이 해당 Subnet에 이런저런 자원(RDS, Elasticache 등...)을 붙일 수 있다. 개념상 A는 Subnet을 공유해 주었지만, B가 관리하는 자원을 A가 쓸 수 있게 된 것이므로, 네트워크를 공유하여 자원도 공유하게 해주는 기능이라 볼 수 있다.

* 참조
  + [AWS VPC Sharing 가이드 문서](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-sharing-share-subnet)

Subnet 같은 경우에는 같은 Organization 아래에 있는 계정들(Principal로 정의)과만 공유 가능하다.  
관리의 분리 차원에서 하나의 워크로드 구축 시 인프라팀과 데이터베이스팀 등으로 구분하여 자원을 운영할 수 있다.

## Usecase

- DevOps 팀의 Kubernetes 인프라에 개발팀 AWS 계정의 DB를 연결하기 위해 활용
- DevOps 팀은 개발팀에게 Kubernetes가 동작 중인 Subnet을 공유, 개발팀은 그 Subnet에 DB를 생성
- SG 설정으로 해당 Shared Subnet의 traffic만 DB에 접근 허용
- Kubernetes에 배포된 개발팀의 앱은 개발팀의 DB의 endpoint를 참조
- DB의 종류, 크기, 성능은 개발팀에서 관리(DB의 관리 책임은 개발팀에게 있음)
- 단, DB에 직접 접속하기 위해서는 DevOps 팀의 네트워크 아래에 있는 bastion 경유 필요
