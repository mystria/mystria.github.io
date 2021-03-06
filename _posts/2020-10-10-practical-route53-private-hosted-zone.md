---
layout: post
title:  "Route53의 Private Hosted Zone 활용"
date:   2020-10-10 24:00:00 +0900
categories: AWS Route53
comments: true
---

# Private Subnet에 구축된 서버들
외부로 노출될 필요가 없거나 되어선 안되는 서버/로드밸런서는 Private Subnet[^PrivateSubnet]에 구축하여 외부에서의 접속을 막는다.  
그렇다면 Private Subnet의 다른 서버들이 Private Sunbnet의 서버/로드밸런서에 접속하려면 어떻게 해야할까?  
역시 기본은 DNS를 통해 IP를 알려주는 것이다. 그러나 이 서버/로드밸런서들은 Public에 노출이 되지 않기 때문에 자체적인 DNS, 즉, Private DNS가 필요하며, Route53에서 이를 제공한다. 바로 Private Hosted Zone.  
이 Private DNS는 우리끼리만 쓰는 것이므로 어떠한 이름도 허용된다. 원한다면 www.google.com으로 지어도 된다.  
이렇게 정의한 Private DNS는 Public DNS 보다 우선 순위가 높기 때문에 Private Subnet의 서버들이 목적지를 조회(resolve) 할 때 우선적으로 검색하게 된다.  
이 처럼 재미있는 활용방법이 있는 Private DNS를 이용하여 서비스를 구축해보자.

## Private Hosted Zone
- 정의: 하나 이상의 VPC 내에 있는 도메인과 그 하위 도메인에 대하여 Amazon Route 53의 DNS 쿼리 응답 정보가 담긴 컨테이너
- 즉, VPC 안에서만 통용되는 DNS
- https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zones-private.html
- Private Hosted Zone을 사용하려면 Amazon VPC에 설정 활성화 필요(기본으로 활성화 되어있음)
  - enableDnsHostnames: true
  - enableDnsSupport: true
- 활용
  - Micro Service 처럼 여러개의 서비스를 구현하고 내부적으로 엮어야 할 경우, 내부용 이름을 지어 줄 수 있음
  - 당연히 domain name으로 물리적인 주소를 덮었기 때문에 호출하는 쪽의 수정없이 호출되는 쪽만 변경 가능
  
## 동일한 hostname으로 internet-facing 로드밸런서와 internal 로드밸런서의 동시 운용
- 로드밸런서를 외부용 / 내부용으로 분리하여 활용
  - 정말 예외적인 경우이겠지만, 하나의 서버에 여러 종류의 로드밸런서를 둘 수 있음, internet-facing과 internal 두 가지
    - 보통은 여러 정치적/기술적/인간적인 사연으로 인해 이런 복잡한 경우가 발생 (호출될 서비스의 hostname이 중요한 경우 등)
    - internal은 VPC 내부에서 통신하지만, internal-facing의 경우 traffic이 public IP로 나갔다 들어오기 때문에 미세한 성능차이가 있을지도? 기술이 좋아서 없겠지?
  - 이 로드밸런서들은 target을 하나의 ASG(Auto Scaling Group)을 가르킴
  - 그리고 이 둘을 똑같은 hostname으로 설정 하여 하나의 endpoint인 것처럼 운용 가능
- 이 로드밸런서들을 Route53에 연결
  - Public Hosted Zone은 그냥 평범하게 설정
  - Private Hosted Zone은 region과 VPC를 추가로 설정

## VPC Peering 너머의 DNS
- 기본적으로 Private Hosted Zone은 VPC Peering 너머로 공유가 안됨
- Public Hosted Zone는 공유되어 조회 가능
  - ~~Enabled 해줘야 하는건지? 그냥 되는지? 확인 필요~~
  - Public Hosted Zone에 Private Subnet의 internal 로드밸런서를 정의하면 건너편 VPC에서 조회되어 Private IP를 반환
    - Private IP를 반환하므로 VPC peering이 되어 있다면 접근 가능
    - 그러나 신기하게도 같은 VPC에서는 이 Public Hosted Zone을 통해 Private IP가 조회 안됨, Public IP가 조회 됨, Private IP가 아니기 때문에 Private Subnet 내부로 접근 불가
- Private Hosted Zone을 별도로 공유 설정 해야 함
  - Private Hosted Zone을 정의할 때 VPC를 2개 추가
    - 각 VPC가 모두 해당 Private Hosted Zone을 조회
    - https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zone-private-associate-vpcs.html
    - (다른 계정의 VPC라면) https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zone-private-associate-vpcs-different-accounts.html
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
    - 이 방법을 쓰지 않는 이유는 운영적인 측면으로 다른 VPC에서 일어나는 일은 나의 R&R 밖의 일일 수 있기 때문에 나도 모르게 내 Private Hosted Zone의 target이 끊어질 수 있기 때문

## 정리
- Private Hosted Zone만 두고 보면 그냥 평범한 기능
- 변칙(?)적으로 운용하면 다양한 네트워크 아키텍처를 구성 할 수 있음
- 보안과 운영을 살리기 좋은 기능


[^PrivateSubnet] Private Subnet은 명시적으로 정의된 기능은 아님, Subnet의 routing table에 IGW(Internet Gateway)가 정의되지 않으면 Private Subnet으로 볼 수 있다. 단, ELB의 경우 internal과 internet-facing으로 구분되는데, internal을 선택하면 Private Subnet처럼 동작한다.

