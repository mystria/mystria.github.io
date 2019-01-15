---
layout: post
title:  "금주의 AWS 실패사례 - ElasticBeanstalk Environment의 신규 App배포 실패"
date:   2018-12-14 18:00:00 +0900
categories: AWS ElasticBeanstalk CloudFormation Failure
comments: true
---
# ElasticBeanstalk에서 새 버전의 App을 배포할때 에러 발생
ElasticBeanstalk(이하 EB)의 Environment(이하 Env)는 하나의 웹앱을 띄우기 위해 완전한 인프라를 구축해 준다.  
그래서 개발자는 앱 개발에만 집중하고, 간편하게 배포 할 수 있다.  
그런데 갑자기 앱 배포가 실패하는 일이 발생하는데...  

## 증상
* Env의 업데이트 실패
  + EB의 Env에서 앱을 업데이트 하였는데 한참 응답없이 멈춰있더니 에러라면서 앱 업데이트가 실패
  + Logs를 받으려해도 역시 한참 무응답 후 실패
  + 보통 이렇게 꽉 막힌 상황에서는 Env를 새롭게 빌드하면 해결되는 경우가 많다.(Immutable Infra, EB의 장점)  
  + 그런데 Rebuild Environment도 한참 프리징 되더니 실패

## 에러 메시지 분석
EB Env의 event에서는 확실한 원인을 표시해주지 않는다. EB도 사실은 CloudFormation의 한 종류이므로, console의 CloudFormation 메뉴로 가서 update를 실패한 stack을 찾아 event를 확인 해 봐야 한다.  
* AWSEBUpdateWaitCondition****** 을 생성하다가 timeout
  > CloudFormation에서 The following resource(s) failed to create: [AWSEBInstanceLaunchWaitCondition].
  + EB의 wait condition은 무엇이고, 왜 생성 도중 실패한 것인가?

## 해결책 조사
Google에서 이것저것 많이 찾아보고 다녀도 뾰족한 답변을 구하지 못했는데, 애초에 이런 일이 잘 발생하지 않는 듯 했다.  
하지만, AWS forum에서 힌트를 얻을 수 있었다.  
https://forums.aws.amazon.com/thread.jspa?threadID=113227#jive-message-412668
* 요약
  > The VPC is not configured correctly. It does not have a NAT instance running through which the ec2 instance can communicate with the internet.  
  > VPC 설정이 옳바르지 않다. EC2 인스턴스가 인터넷과 통신하게 해줄 NAT 인스턴스가 없다.  
* 풀이
  + Private Subnet에 위치하는 EC2 가 인터넷에 연결되기 위해서는 NAT 인스턴스 또는 NAT Gateway 가 필요
    - NAT Gateway는 동일하게 NAT 기능을 하는 관리형 서비스
    - 단, 외부에서 접근하기 위한 Bastion 서버 역할을 하지 못함
  + 즉, VPC의 네트워크가 문제가 되면 그와 관련된 CloudFormation(ElasticBeanstalk도 여기 포함)이 제대로 수행되지 않음

## 해결
* NAT 인스턴스 문제
  + 확인 결과 하나의 NAT 인스턴스가 다운
  + 정상 동작하는 Env를 확인해 보니 NAT가 정상 동작하는 Subnet에서 EC2 인스턴스가 동작 중 이었음
* NAT 인스턴스 재생성
  + 간단하게 새로 띄우고, Route Table 재설정
* 생각꺼리
  + CloudFormation 수행 시 EC2가 인터넷에 접근해야 하는 이유가 뭘까? 
    - Artifact를 받기 위해 S3에 접근할 때 인터넷 통신을 하는지?
    - EC2가 Amazon network 내에 있더라도 NAT가 없으면 이런저런 traffic 조차 완전히 차단되는지?

