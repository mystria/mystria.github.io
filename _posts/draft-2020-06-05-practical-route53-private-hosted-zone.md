---
layout: post
title:  "Route53의 Private Hosted Zone 활용"
date:   2020-06-05 24:00:00 +0900
categories: AWS Route53
comments: true
---

# Private Subnet에 구축된 서버들
외부로 노출될 필요가 없거나 되어선 안되는 서버/로드밸런서는 Private Subnet에 구축하여 외부에서의 접속을 막는다.  
그러나 Public Subnet이나 마찬가지로 Private Subnet의 다른 서버들이 Private Sunbnet의 서버/로드밸런서에 접속하려면 어떻게 해야할까?  
역시 기본은 DNS를 통해 IP를 알려주는 것이다. 그러나 이 서버/로드밸런서들은 Public에 노출이 되지 않기 때문에 자체적인 DNS, 즉, Private DNS가 필요하다.
Route53에서는 이를 제공한다. 이 Private DNS는 우리끼리만 쓰는 것이므로 어떠한 이름도 허용된다. 원한다면 www.google.com으로 지어도 된다.  
이렇게 정의한 Private DNS는 Public DNS 보다 우선 순위가 높기 때문에 Private Subnet의 서버들이 목적지를 조회(resolve) 할 때 우선적으로 검색하게 된다.  
이 처럼 재미있는 활용방법이 있는 Private DNS를 이용하여 서비스를 구축해보자.

## Private Hosted Zone
- 정의: 하나 이상의 VPC 내에 있는 도메인과 그 하위 도메인에 대하여 Amazon Route 53의 DNS 쿼리 응답 정보가 담긴 컨테이너
- 즉, VPC 안에서만 통용되는 DNS
- https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zones-private.html
- Private Hosted Zone을 사용하려면 Amazon VPC에 설정 활성화 필요(기본으로 활성화 되어있음)
  - enableDnsHostnames: true
  - enableDnsSupport: true
  
## 동일한 hostname으로 internet-facing 로드밸런서와 internal 로드밸런서의 동시 운용
- 로드밸런서가 외부용 / 내부용으로 분리되어 있음
- 이 로드밸런서를 Route53에 연결하고자 함

## VPC Peering 너머의 DNS
- Public Hosted Zone는 공유되어 조회 가능
  - Enabled 해줘야 하는건지? 그냥 되는지? 확인 필요
  - Public Hosted Zone에 정의한 internal 로드밸런서가 조회되어 Private IP를 반환
    - Private IP를 반환하므로 VPC peering이 되어 있다면 접근 가능(20201004)
    - 신기하게도 같은 VPC에서는 이 Public Hosted Zone을 통해 Private IP가 조회 안됨
- Private Hosted Zone도 공유 가능
  - Private Hosted Zone을 정의할 때 VPC를 2개 추가
    - 각 VPC가 모두 해당 Private Hosted Zone을 조회
    - https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zone-private-associate-vpcs.html
    - (다른 계정의 VPC) https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zone-private-associate-vpcs-different-accounts.html
    - https://aws.amazon.com/ko/premiumsupport/knowledge-center/private-hosted-zone-different-account/
      ~~~ ssh
        # profile 잘 확인하기

        # 1. Guest VPC에게 권한주기
        $ aws --profile hostedZoneOwner route53 create-vpc-association-authorization --hosted-zone-id ZONEID_FROM_ROUTE53 --vpc VPCRegion=eu-central-1,VPCId=vpc-guest_vpc_id

        # 2. Guest가 VPC 연결 요청하기
        $ aws --profile hostedZoneGuest route53 associate-vpc-with-hosted-zone --hosted-zone-id ZONEID_FROM_ROUTE53 --vpc VPCRegion=eu-central-1,VPCId=vpc-guest_vpc_id

        # 3. Guest의 VPC의 권한 삭제
        $ aws --profile hostedZoneOwner route53 delete-vpc-association-authorization --hosted-zone-id ZONEID_FROM_ROUTE53 --vpc VPCRegion=eu-central-1,VPCId=vpc-guest_vpc_id
      ~~~
  - Private Hosted Zone을 공유하지 않고 각각 새로 생성
    - 기존 대상 Private Hosted Zone에다 다른 계정의 VPC를 추가하지 않고, 추가하려는 VPC가 있는 계정에 Private Hosted Zone을 추가
    - 이 Hosted Zone에 추가되는 record에 A type Alias 또는 CNAME을 추가하면 됨
    - Alias는 다른 계정이라 자동 검색이 되지않으므로 직접 대상 입력 필요
    - 참고로, ELB의 주소는 public DNS에서 검색됨
ㄴ