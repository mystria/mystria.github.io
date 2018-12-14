---
layout: post
title:  "금주의 AWS 실패사례 - EC2의 key pair를 바꿔보자"
date:   2018-07-18 20:00:00 +0900
categories: AWS ElasticBeanstalk CloudFormation
comments: true
---
# ElasticBeanstalk에서 새 버전의 App을 배포할때 에러

NAT 서버가 죽으니까 앱 재배포도 에러나고, Rebuild도 에러나네.
CloudFormation에서 The following resource(s) failed to create: [AWSEBInstanceLaunchWaitCondition]. 에러가 나는데,
AWSEBUpdateWaitCondition****** 단계에서 멈추다가 timeout

왜일까?

일단 해결책의 힌트는
https://forums.aws.amazon.com/thread.jspa?threadID=113227#jive-message-412668
여기서 얻음
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/vpc-bastion-host.html
어쨋든 VPC의 네트워크가 문제가 되면 그와 관련된 CloudFormation(ElasticBeanstalk도 여기 포함)에 문제가 있다.
실제로 같은 VPC였지만 NAT 정상 동작하는 Subnet은 문제 없었음

CloudFormation 수행 시 해당 Subnet에 접근해야 하는 어떤 트래픽이 오고 가는걸까?
Private Subnet에서도 NAT가 없이 인터넷은 될텐데...


