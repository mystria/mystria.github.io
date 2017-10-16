---
layout: post
title:  "AWS Elastic Load Balancer와 SSL인증서"
date:   2017-10-13 14:00:00 +0900
categories: AWS Tip ELB ElasticBeanstalk SSL
comments: true
---
# AWS ELB와 EB 미립자 팁
1. [ElasticBeanstalk의 ELB찾기](#elasticbeanstalk의-elb찾기)
2. [ELB에 SSL인증서 붙이기](#elb에-ssl인증서-붙이기)
2. [ELB 보안, 인증서 정책 바꾸기](#elb-보안-인증서-정책-바꾸기)

## ElasticBeanstalk의 ELB찾기
* ElasticBeanstalk(이하 EB)에서의 ELB
  + Environment의 Capacity가 Single Instance말고 Load Balanced일때 만 활성화
  + Configuration의 Network Tier에서 Load Balancer 설정 가능
  + 설정할 수 있는 것
    - SSL Certificate
    - Sticky Session공
    - Health Check 등..
  + 그러나 보다 구체적으로 설정하고 싶을때, EC2항목의 Load Balancing > Load Balancers로 가서 해당 ELB를 찾아보자
    - 찾기 힘듦(AWS Admin Console의 문제점.. 사용성이 떨어짐)
* Load Balancer 찾기
  1. Instances에서 EB의 Environment이름으로 Instance찾기
  2. 찾은 Instance의 InstanceId를 통해 Auto Scaling > Auto Scaling Group(이하 ASG)에서 ASG찾기
  3. 찾은 ASG의 Details정보에서 Load Balancers의 ID확보
  4. 해당 ID로 Load Balancers에서 ELB찾기 성공


## ELB에 SSL인증서 붙이기
* SSL 등록
  + 현재 SSL처리는 Load Balancer(이하 LB)가 전담
    - LB 뒷단은 일단은 (보안상 안전할 것으로 보고) 평문 통신
    - EC2에서도 SSL처리 하기 원할 경우 Proxy 설정 등에 수동으로 script를 넣어 적용시켜야 함
  + 예전에는 EC2 > Load Balancers > Load Balancing에서 해당 LB 선택 후 SSL 인증서를 업로드해야함
    - 즉 ELB에 SSL을 개별적으로 붙여야 함
    - 대상 Load Balancer선택 후 Listener에서 SSL Certificate 변경(Change)으로 업로드/변경
    - 일단 업로드 된 것은 EB에서도 Load Balancer설정 시 Configuration에서 적용 가능함
  + 현재는 Amazon Certificate Manager에서 관리 기능 지원
    - 무료 SSL 인증서를 제공하기도 하고, 업로드도 지원
    - 여기서 관리되는 인증서를 ELB에서 사용 가능
    - 단, Load Balancer에서 업로드 한 것은 Certificate Manager에서 관리 안됨
* Load Balancer에 적용
  + EB의 경우 Environment의 Configuration에서 Load Balancer에서 적용 가능
  + 개별 Load Balancer를 찾아 Listener tab에서 수정 가능
* Load Balancer의 Listener란?
  + 참고로 Elastic LB는 HTTP/HTTPS를 지원하는 Application LB, TCP를 지원하는 Network LB, 그리고 기존의 Classic LB로 구분
  + Classic LB
    - EB에서는 아직도 Classic LB사용
    - LB로 들어온 Protocol과 Port를 bind된 Instance의 Protocol과 Port로 라우팅
  + Application LB, Network LB팅
    - 언제 부터 생긴거지... [AWS ELB][aws-elb]
    - Application은 Layer 7로 고급 라우팅 기능 제공, Network는 Layer 4로 고성능 분배에 적합
    - Rule기반으로 지정된 Target Group으로 라우팅

## ELB 보안, 인증서 정책 바꾸기
* 공식홈페이지 참조 : [AWS Classic LB Listener][aws-https-listner1], [AWS Application LB Listener][aws-https-listner2]
* 보안 정책 수정하는 위치
  0. AWS Admin Console은 정말 사용자 친화적이지 않음(익숙해지면 불편하진 않음)
  1. 일단 Load Balancer 찾기
  2. Listeners 탭
    - Classic LB의 경우, Load Balancer Protocol이 HTTPS또는 SSL일 경우 Cipher항목 변경(Change)가능
    - Application/Network LB의 경우, Listener 선택 후 수정(Action > Edit)
  3. 보안 정책 선택
    - Classic LB의 경우, Predefined와 Custom Security Policy 선택 가능
    - Application/Network LB의 경우, Predefined된 Security Policy만 지원
  4. 적절히 선택 후 적용
* 보안 정책을 수정해야 하는 이유
  + TLS는 SSL3.0을 개선한 국제 표준 프로토콜, 그러나 하위 호환성을 위해 클라이언트 요청에 의해 다운그레이드 가능
  + TLS1.0과 SSL3.0에는 취약점이 있다고 함(자세한 내용은 [인터넷 검색][tls-ssl-vulnerability])
  + TLS1.0과 SSL3.0의 보안 취약점을 이용하기 위해, TLS의 버전을 다운그레이드 하게끔 유도하는 공격[KISA보도자료][poodle]
  + 앗싸리 TLS1.0과 SSL3.0을 비활성화 하길 권장
  + 오래전에 생성한 ELB의 경우 어떤 암호화 방식을 지원하는지 보안 정책을 확인해야 함

[aws-elb]: https://aws.amazon.com/elasticloadbalancing/details/#compare
[aws-https-listner1]: http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-https-load-balancers.html
[aws-https-listner2]: http://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html
[poodle]: https://www.krcert.or.kr/data/trendView.do?bulletin_writing_sequence=22128
[tls-ssl-vulnerability]: https://www.kb.cert.org/vuls/id/864643
