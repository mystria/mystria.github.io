---
layout: post
title:  "금주의 AWS 실패사례 - CloudFormation을 이용한 Elastic Load Balancer의 Listener 설정"
date:   2018-04-10 21:00:00 +0900
categories: AWS ELB CloudFormation
comments: true
---
# Deprecated된 aws:elb:loadbalancer 설정들을 대체하기
블라블라  

## 시밤
블러러  
  * EC2 에서 HTTPS 프로토콜 처리 방법
    1. 대상 instance에 SSL 인증서 파일을 넣는다.
    2. 대상 instance의 proxy에서(보통 apache 또는 nginx사용) 설치된 인증서를 이용하여 HTTPS 프로토콜을 처리한다.
https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-properties-elasticbeanstalk-configurationtemplate-configurationoptionsetting.html
https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/ebextensions-optionsettings.html
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.elb.html
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options-general.html#command-options-general-elblistener-listener

## 샘플

          {
            "Namespace": "aws:elb:listener:443",
            "OptionName": "InstancePort",
            "Value": { "Fn::If": [ "UseHttps", "80", "443" ] }
          },
          {
            "Namespace": "aws:elb:listener:443",
            "OptionName": { "Fn::If": [ "UseHttps", "SSLCertificateId", "InstancePort" ] },
            "Value": { "Fn::If": [ "UseHttps", { "Ref": "CertificateArn" }, "443" ] }
          },
