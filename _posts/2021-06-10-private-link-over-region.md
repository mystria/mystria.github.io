---
layout: post
title:  "다른 Region에서 PrivateLink의 Endpoint를 연결하기"
date:   2021-06-10 20:00:00 +0900
categories: AWS PrivateLink Endpoint
comments: true
---

# 다른 Region에서 PrivateLink의 Endpoint를 연결하기
PrivateLink Endpoint로 연결하려는 서비스가 우리 서비스와 다른 region에 배포되어 있다면?

## PrivateLink
* PrivateLink를 쓰면 좋은 점
  + 서비스 간 연결이 외부로 노출 안됨(내부망에서만 트래픽 움직임)
  + 서비스 제공자가 연결 제어 가능 (단방향 설정 같은)
  + 그래서 보안/성능적으로 좋음
  + https://aws.amazon.com/ko/privatelink

## Region 제약
PrivateLink는 region기반 서비스이다. 다양한 장점에도 불구하고 multi-region한 서비스를 구성할 때 약간의 추가 작업으로 불편해지게 되는데, region에 종속된 VPC간 네트워크 연결이므로 당연한 것일지도 모르겠다. 그리고 그렇기 때문에 VPC peering으로 해결 할 수 있다.  
Best practice는 확인 못했지만, 이론적으로 생각할 수 있는 구성이고 실제로도 동작한다. 어떤 간단한 설정으로 VPC peering 없이 해결할 수 있는지는 따로 확인 필요 / 또는 향후 AWS에서 지원할지도 모른다.

## 작업 방법
* Endpoint 생성
  + PrivateLink의 Endpoint를 생성하려면 서비스(Endpoint Service)가 제공되는 region과 동일한 region에서 작업해야 함
  + PrivateLink가 존재하는 region A에 VPC A를 생성하고, 이 VPC에 Endpoint 생성
    - 이때 VPC만 생성하면 되는 것이 아니라, Subnet(AZ기반) 등을 적절히 생성해야 한다. 특히 AZ를 서비스와 맞추는게 바람직하다.
* VPC peering
  + 우리 서비스가 동작 중인 region B의 VPC B를 VPC A와 peering하여 문제를 해결
    - PrivateLink가 제시하는 장점 중의 하나가 네트워크 구성의 간소화임
    - Peering을 할 때 반드시 route table도 설정 필요

## Endpoint에 URL이 적용되어야 한다면
* MongoDB Atlas의 경우
[MongoDB Atlas의 경우](https://docs.atlas.mongodb.com/security-private-endpoint/) 자체 DNS를 통해 CNAME(Canonical Name)으로 endpoint를 읽기 좋게 감싸서 제공해주는데, 이때는 VPC peering만 해도 제공되는 URL을 타고 목적 서비스의 IP에 도달할 수 있다.  
확실한 내부 구성은 잘 모르겠으나, MongoDB VPC의 private hosted zone에서 Endpoint를 가리키는 URL을 설정해주어 peering된 VPC의 DNS를 탈 수 있는 것으로 보인다.
* 일반적인 Endpoint의 경우

    | DNS names |
    |---|
    | vpce-00000xxxxx00000cc-aaaabbbb.vpce-svc-00000xxxxx00000aa.us-east-1.vpce.amazonaws.com (Z7XXXXXUULQXV) |
    | vpce-00000xxxxx00000cc-aaaabbbb-us-east-1a.vpce-svc-00000xxxxx00000aa.us-east-1.vpce.amazonaws.com (Z7XXXXXUULQXV) |
    | vpce-00000xxxxx00000cc-aaaabbbb-us-east-1b.vpce-svc-00000xxxxx00000aa.us-east-1.vpce.amazonaws.com (Z7XXXXXUULQXV) |
    | internal-api.service.com (Z069XXXXXM0LMA0004WUC) |

  + Endpoint-specific DNS hostname
  Endpoint를 생성하고 Endpoint 세부정보의 DNS name을 보면 여러개의 AWS에서 제공했을 법한 읽기 힘든 URL을 볼 수 있는데 우리는 이를 통해 서비스에 접근할 수 있다.  
  예) vpce-00000xxxxx00000cc-aaaabbbb.vpce-svc-00000xxxxx00000aa.us-east-1.vpce.amazonaws.com (Z7XXXXXUULQXV)
  + Private DNS name
  Endpoint Service에서 서비스의 DNS 설정을 통해(어떻게 설정하는지는 확인 필요) URL과 동일하게 Endpoint를 제공해주는 경우에는 위에 언급한 Atlas와 같이 해당 URL을 통해 바로 접근이 가능하다. 
  예) internal-api.service.com (Z069XXXXXM0LMA0004WUC)

## 연결해보기
* DNS 확인
이 중 AZ가 명시되지 않은(아마도 LoadBalancer인 듯한) URL을 선택하여, VPC A에서 접속해보면 AZ교대로 접속이 되는것을 알 수 있다. AZ가 명시된 다른 것을 써도 되지만 특별한 제약이 없는 한 AZ가 없는 것을 쓰는게 자동으로 골고루 AZ routing이 된다는 점에서 좋을 것 같다.  
이 Endpoint DNS들은 확인해보면 마치 VPC A에 위치한 것 처럼 internal IP로 연결되어 있다.  
  >[ec2-user@ip-10-0-2-12 ~]$ ping internal-api.service.com
PING internal-api.service.com (10.0.1.62) 56(84) bytes of data.

* 직접 DNS 적용하기
혹시 Endpoint에 SSL이 적용되어 있다면 거기에 해당하는 domain을 사용해야한다. MongoDB Atlas처럼 직접 domain에 싸서 주거나, Private DNS name을 사용한다면 그냥 사용하면 되지만, 이런 세심한 배려가 없다면 추가 작업이 필요하다.  
  + Route53에서 VPC B에 종속된 private hosted zone을 생성
  + A type의 record에 Alias(Endpoint-specific DNS hostname)로 엮어 생성
    - Alias로 선택하면 region A와 VPC A를 지정하여 Endpoint의 URL을 찾을 수 있음
  + 이 가상 record에 SSL에 해당하는 URL을 지정해주면 된다.

* Endpoint의 옵션 변경
Private DNS name을 제공하는 경우에는 우리의 Endpoint의 Private DNS names enabled도 활성화 해야하며, 그 URL을 통해 VPC A에 존재하는 instance처럼(VPC에서 public ip기반 domain활성화 해주는 그것) 접속이 가능하다.

## Private DNS name의 문제
그러나 Private DNS name은 설정된 VPC A에서만 가능하고 이를 peering한 VPC B에서는 연결이 되지 않는 문제가 있다. 이상하게도 VPC B에서는 이 private DNS name을 찾을(resolve할) 수 없다.  
>[ec2-user@ip-10-0-2-12 ~]$ ping internal-api.service.com
ping: unknown host internal-api.service.com

이를 해결하기 위해 위 "직접 DNS 적용하기" 처럼, 직접 VPC B에 private hosted zone을 구성해주어야 한다.   
\* 참고로 VPC Peering의 DNS Setting으로도 시도해봤지만 해결이 되지 않았음  

| DNS Settings |
|--------------|
| Requester VPC (vpc-0000ddcxxxxx0ef4b) peering connection attributes: <br/> DNS resolution from accepter VPC to private IP: Disabled |
| Accepter VPC (vpc-0000878xxxxx8952d) peering connection attributes:	<br/> DNS resolution from requester VPC to private IP: Enabled |

## 만약 VPC peering이 없다면?
주의할 점은 만약 VPC A와 B 간에 VPC peering이 없으면, (당연하게도) Endpoint의 internal IP를 얻어도 해당 IP로 접근이 불가능하다. 
VPC A와 B를 peering 할 때, 둘다 Route table을 업데이트하여 VPC간 IP대역을 서로 route해준다는 사실을 잊지 말자.

## 참조
[PrivateLink의 Interface Endpoint의 설명](https://docs.aws.amazon.com/ko_kr/vpc/latest/privatelink/vpce-interface.html#vpce-private-dns)
