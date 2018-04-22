---
layout: post
title:  "금주의 AWS 실패사례 - CloudFormation을 이용한 Elastic Load Balancer의 Listener 설정"
date:   2018-04-10 21:00:00 +0900
categories: AWS ELB CloudFormation
comments: true
---
# Deprecated된 aws:elb:loadbalancer 설정들을 대체하기
CloudFormation으로 ElasticBeanstalk 환경을 구축하는데, Load Balancer 설정을 하려했다.  
예전에 만들어둔 template에서는 aws:elb:loadbalancer라는 namespace의 option_settings 값을 설정하면 간편(?)하였는데, 아마도 좀 더 정교한 설정을 위해 이 설정은 deprecated되었다.  
정교해진만큼 설정해야 할 option도 많아졌다.  
그 중 SSLCertificateId의 값을 설정할 때 문제가 발생했다.

## Elastic Load Balancer에 SSL인증서 설정
Elastic Load Balancer(이하 ELB)의 역할은 물론 네트워크 입력을 분산시켜 처리하도록 해주는 것이다. 하지만 ELB는 한 가지 역할이 더 있는데, 바로 SSL인증서 처리이다.  
  * Load Balancer에서

## CloudFormation으로 ElasticBeanstalk를 생성

## OptionSettings를 설정하여 ElasticBeanstalk의 세부 설정

## aws:elb:loadbalancer -> aws:elb:listener:*port*
  * loadbalancer Namespace가 deprecated됨
    + 자세한 이유는 모르겠지만, Load Balancer가 좀 더 다양해지고 구체적이 되어 관리 포인트도 증가하여 설정도 세분화 한것으로 추정
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
