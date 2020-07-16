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
- dd
  
## 동일한 hostname으로 internet-facing 로드밸런서와 internal 로드밸런서의 동시 운용
- 로드밸런서가 외부용 / 내부용으로 분리되어 있음

## VPC Peering 너머의 DNS
- Public Hosted Zone는 공유되어 조회 가능
  - Enabled 해줘야 하는건지? 그냥 되는지? 확인 필요
  - Public Hosted Zone에 정의한 internal 로드밸런서가 조회되어 Private IP를 반환
    - 신기하게도 같은 VPC에서는 이 Public Hosted Zone을 통해 Private IP가 조회 안됨
- Private Hosted Zone도 공유 가능
  - Private Hosted Zone을 정의할 때 VPC를 2개 추가하면 됨
  - https://aws.amazon.com/ko/premiumsupport/knowledge-center/private-hosted-zone-different-account/

