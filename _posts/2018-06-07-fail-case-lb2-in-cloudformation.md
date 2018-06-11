---
layout: post
title:  "AWS Application Load Balancer와 EB CloudFormation"
date:   2018-06-07 22:00:00 +0900
categories: AWS CloudFormation ELB ElasticBeanstalk
comments: true
---
# AWS의 Application Load Balancer를 EB용 CloudFormation으로 정의할 때 만난 문제점
AWS의 EC2는 클라우드에서 동작하는 일종의 "서버", 클라우드는 이런 서버를 무수히 확장(Scale-out)하여 대용량 트래픽에 대응 가능하다.  
AWS에서는 Auto Scaling Group(서버 묶음)과 이들에게 트래픽을 분배하는 Load Balancer를 제공한다.  
Elastic Load Balancer라 불리는 이것은 Application Load Balancer(이하 ALB)와 Classic Load Balancer(이하 LB)로 구분 된다.  
이름에서 보다시피 예전에는 classic만 존재하였으나 모니터링 metric을 몇 개 더 추가하고, 비용을 줄여 ALB를 출시하였다.  
(아마 Serverless 환경을 노린듯)  
AWS ElasticBeanstalk를 사용하면 Load Balancer를 구축할 수 있는데, 이때 ALB와 LB를 선택 가능하다.
그리고 CloudFormation을 이용하여 ElasticBeanstalk를 생성할 때, 이러한 설정들을 OptionSettings로 전달해야 한다.
여기에 함정이 숨겨져 있었다.

## 발단
ElasticBeanstalk(이하 EB)를 CloudFormation(이하 CF)으로 생성, 이때 Load Balancer를 classic과 application중 선택 가능  
CF로 전달되는 Parameters에 따라 classic과 application을 선택할 수 있게하고, listener를 HTTP와 HTTPS를 선택할 수 있게 했음  
  * Classic
    + OptionSettings에 "aws:elb:listener"를 정의해야 함
    + OptionSettings에 "aws:elb:listener:443"를 정의해야 함
  * Application
    + OptionSettings에 "aws:elbv2:listener"를 정의해야 함
    + OptionSettings에 "aws:elbv2:listener:443"를 정의해야 함

이때, CF template을 실행할 때 문제가 생김

## 증상
완성된 CF template를 실행시키면 아래와 같은 메시지
~~~
  Configuration validation exception: You must specify an SSL certificate to configure a listener to use HTTPS. (Service: AWSElasticBeanstalk; Status Code: 400; Error Code: ConfigurationValidationException; Request ID: b8xxxxx3-d5b8-48b2-8087-7xxxxxxxxxx3)
~~~
HTTPS listener를 쓰기 위해선 SSL인증서를 설정해야 한다는 뜻

## 원인
CF의 Parameters에 따라 ALB/LB, HTTP/HTTPS를 선택하게 하기 위해서는 아래의 예제와 같은 것들을 설정해야 함  
  * HTTP의 ALB/LB ListenerEnabled
    ~~~ json
      {
        "Namespace": "aws:elb:listener",
        "OptionName": "ListenerEnabled",
        "Value": { "Fn::If": [ "UseHttp", true, false ] }
      },
      {
        "Namespace": "aws:elbv2:listener:default",
        "OptionName": "ListenerEnabled",
        "Value": { "Fn::If": [ "UseHttp", true, false ] }
      },
    ~~~
  * HTTPS의 ALB/LB ListenerEnabled
    ~~~ json
      {
        "Namespace": "aws:elb:listener:443",
        "OptionName": "ListenerEnabled",
        "Value": { "Fn::If": [ "UseHttp", false, true ] }
      },
      {
        "Namespace": "aws:elbv2:listener:443",
        "OptionName": "ListenerEnabled",
        "Value": { "Fn::If": [ "UseHttp", false, true ] }
      },
    ~~~

  CF의 조건문(Conditions)는 그다지 강력하지 않기 때문에 특정 조건에서만 Option값을 넣거나 뺄 수 없음  
  그래서 일단 필요한 값을 다 넣은다음 OptionName이나 Value를 조절해야 함  
  이 과정에서 아래와 같은 값도 미리 넣어놔야 함, 왜냐하면 위의 ListenerEnabled가 들어가 있는 이상 "없음" 상태가 아니라 존재하지만 "비활성화" 된 것이기 때문  
  * HTTP/HTTPS의 protocol 설정
    ~~~ json
      {
        "Namespace": "aws:elb:listener:443",
        "OptionName": "ListenerProtocol",
        "Value": "HTTPS"
      },
      {
        "Namespace": "aws:elbv2:listener:443",
        "OptionName": "Protocol",
        "Value": "HTTPS"
      },
    ~~~

    여기서 문제가 생김, 443포트에 대한 protocol을 HTTPS로 했기 때문에 SSL Certificate가 필요하단 것  
    443포트이지만 HTTP로 지정해야 함

## 해결책
  * AWS::NoValue
    + AWS에서 사전 정의한 Pseudo 값
    + 안쓰는 곳에 쓰면 "없음"과 같이 될 것 같았지만, *안됨*
      - 다음은 서로 다름
        ~~~ json
          {
            "Namespace": "aws:elbv2:listener:443",
            "OptionName": "Protocol",
            "Value": "AWS::NoValue"
          },
          {
            "Namespace": "aws:elbv2:listener:443",
            "OptionName": "Protocol"
          },
        ~~~
      - 다음과 같이 조건문을 사용해야 함
        ~~~ json
          {
            "Namespace": "aws:elb:listener:443",
            "OptionName": "ListenerProtocol",
            "Value": { "Fn::If": [ "UseHttps", "HTTPS", "HTTP" ] }
          },
          {
            "Namespace": "aws:elbv2:listener:443",
            "OptionName": "Protocol",
            "Value": { "Fn::If": [ "UseHttps", "HTTPS", "HTTP" ] }
          },
        ~~~
  * Certificate의 경우
    + 마찬가지로 AWS::NoValue가 적용되지 않음
    + 아래처럼 SSLCertificateId / SSLCertificateArns 와 같은 OptionName이 처음부터 없던것 처럼 만들어야 함
      ~~~ json
        {
          "Namespace": "aws:elb:listener:443",
          "OptionName": { "Fn::If": [ "UseHttps", "SSLCertificateId", "InstancePort" ] },
          "Value": { "Fn::If": [ "UseHttps", { "Ref": "CertificateArn" }, "443" ] }
        },
        {
          "Namespace": "aws:elbv2:listener:443",
          "OptionName": { "Fn::If": [ "UseHttps", "SSLCertificateArns", "Protocol" ] },
          "Value": { "Fn::If": [ "UseHttps", { "Ref": "CertificateArn" }, "HTTP" ] }
        },
      ~~~
