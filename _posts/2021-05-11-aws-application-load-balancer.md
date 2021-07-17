---
layout: post
title:  "AWS ApplicationLoadBalancer 구조 살펴보기"
date:   2021-05-11 20:00:00 +0900
categories: AWS ALB ASG
comments: true
---

# AWS ApplicationLoadBalancer의 구조를 살표보자
간단한 내용이지만 맨날 Elastic Beanstalk(이하 EB)만 쓰다보니 가물가물해져서 정리함

## ApplicationLoadBalancer - TargetGroup
* ApplicationLoadBalancer(이하 LB)는 AutoScalingGroup(이하 ASG)와 관련되어 있음
* 왜냐하면 ASG에서 상황에 따라 instance 개수를 증감시키게되는데, LB가 이 instance들을 routing해줘야 하기 때문
* 그런데 LB의 설정에는 ASG를 연결/지정하는 부분이 없음 (아마 Classic에서 Application으로 바뀌면서 없어짐 - 확실치 않음)
* LB설정에는 대신 TargetGroup(이하 TG)이라는 것이 존재
* LB는 listener(Protocol/Port별)를 설정할 수 있고, listner는 rule(header, method, ip등 별)을 설정할 수 있음
* 이 rule에서 조건에 따른 TG를 지정하게 되는데,
TG에 가보면 대상 instance 목록을 확인할 수 있고 정보 및 상태가 표시됨(healthy/unhealthy)

* 좀 복잡하지만 정리하자면,
  > ALB - listner - rule - TG - instance
  
  이런 순으로 찾아가게됨

## AutoScalingGroup - TargetGroup
* ASG는 별도의 개체로서 TG를 지정함 (ASG의 Details > Load balancing)
* 즉, auto scaling을 하게되면 그 결과를 TG에게 안내해주고, 
TG가 업데이트되면 LB는 자연스레 최신 대상으로 routing 하게 해줌

## ElasticBeanstalk
* EB에서는 LB 설정과 ASG설정(Capacity)이 간략하게 되어 있어서 이러한 연결고리는 실제 각 instance, LB, ASG를 찾아가서 확인해야 함
* EB는 1set의 LB만 갖고 있는데, 상황에 따라서 위의 ASG와 target group 설정등을 해준다음 추가로 LB를 만들어 사용할 수 있음
  + 주 LB는 외부노출용, 추가 LB는 내부용 같이 이원화된 설정같은 경우

* 참고로 web console에서 EB정보만으로 LB를 찾으려면 좀 힘든데,
tag:elasticbeanstalk:environment-name 또는 tag:elasticbeanstalk:environment-id를 이용해서 찾거나 vpc-id로 찾는 수 밖에 없는 것 같음
* Name이나 DNS name같이 주요 field로는 검색이 어려움
* ASG는 LB의 Name(tag:Name아님)으로 검색하면, 연결된 Load balancing(target group)을 통해 찾을 수 있음

## 참고
* https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/load-balancer-target-groups.html