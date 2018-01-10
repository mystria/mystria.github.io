---
layout: post
title:  "AWS 미립자 팁 - IAM, EC2"
date:   2018-01-10 18:00:00 +0900
categories: AWS EC2 IAM
comments: true
---
# AWS 미립자 팁
해당 포스팅은 AWS Documents에 이미 설명되어 있지만 저만 몰랐던 팁을 정리하였습니다.

## EC2
  * EC2의 KeyPair는 수정 가능함
    + EC2가 Public Key를 갖고 있고, 접속하려는 사람이 해당 Public Key의 Private Key로 인증
    + 즉, Public Key만 변경하면 됨 : 위치는 ~/.ssh/authorized_keys
    + 만약, KeyPair를 잃어버린다면, 해당 EC2의 EBS를 분리하여 다른 EC2에 붙여 해당 파일을 수정하면 해결가능
  * 또한, 여러개의 KeyPair 등록 가능
    + KeyPair를 사용자 별로 나눠 줄 수도 있고, 주기적으로 변경할 수도 있다는 뜻, OpenSource Project로 존재하는 듯
  * 관련정보 : [[Amazon EC2 키 페어](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-key-pairs.html)]

## IAM
  * 관리자급의 Policy를 만들 때 보통 Allow * 함
  ``` json
  {
    "Effect": "Allow",
    "Action": "*",
    "Resource": "*"
  }
  ```
  * 해당 Policy에서 특정 Action만 제외하고 싶다면 Deny를 사용
  ``` json
  {
    "Effect": "Deny",
    "Action": "IAM:*",
    "Resource": "*"
  }
  ```
  * Deny를 사용할 경우 해당 Action은 어떤 경우에도 사용불가, 다른 Policy에서 Allow해줘도 Deny가 우선되어 차단됨
  * NotAction, NotResource, NotPrincipal을 활용할 것
  * Condition으로 해당 Effect를 무효화 할 수 있음
  * 관련정보 : [[IAM 정책 요소 참조](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/reference_policies_elements.html)]
    + 위 웹페이지를 참고하여 다양한 각도로 IAM 관리가 가능
  * 개인적인 정리
    + IAM의 권한은 3단계 : Allow, 중립, Deny
      - Allow는 허가됨
      - 중립은 허가되지 않음
      - Deny는 금지됨
    + Allow와 Deny가 만나면 무조건 Deny됨
      - 그래서 Allow하지 않는다고 Deny하게되면 향후 권한 충돌로 Deny된 권한을 사용할 수 없음
      - Inline Policy로 되어 있을 경우, 권한 차단 원인을 찾기 힘들 수도 있음
    + 최선의 방법은, 필요한 기능만 Allow, 불필요한 기능은 중립, 막아야 하는 것은 Deny
      - 불필요한 기능보다 필요한 기능이 더 많다면, 각각 Allow하기는 힘드니 NotAction, NotResource등으로 예외처리 하는 방식 사용
      - 단, Allow + Not**은 명시적으로 표현하지 않은 권한들도 사용 가능 할 수 있음
      - Deny + Not**은 대상외의 권한을 막지만, 권한을 준 것은 아님
  * 예제: 모든 권한을 갖고 있지만, IAM관련 기능은 본인관련 리소스만 접근가능
  ``` json
  {
    {
      "Effect": "Allow",
      "NotAction": "iam:*",
      "Resource": "*"
    },야
    {
      "Effect": "Allow",
      "Action": "iam:*",
      "Resource": "arn:aws:iam::123456789010:user/${aws:username}"
    }
  }
  ```
