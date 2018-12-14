---
layout: post
title:  "AWS에서 TLS certificate를 적용할 방법 3가지"
date:   2018-07-23 17:00:00 +0900
categories: AWS ELB EC2 CloudFront CertificateManager
comments: true
---
# EC2 Instance에 TLS 인증서를 적용하자
AWS에서 Web Application을 구동시킬 때, 꼭 필요한 것 중 하나가 TLS(SSL) 인증서 적용이다.  
예전에는 TLS 인증서를 적용하여 HTTPS 프로토콜을 사용 할 경우 성능 저하가 있다고도 하였으나, 최근 연구에 따르면 성능 문제는 전혀 없다고 한다.(관련 링크)  
어쨋든 중간자 공격(MITM)도 예방하고, 멋진 웹 사이트를 제작하기 위해 TSL 인증서를 AWS에서 동작하는 서버에 적용하여 보자.  
인증서 구매 및 적용 방법은 여기서 서술하지 않을 예정이다. 더 자세하고 잘 설명해 주는 곳은 많다.  
또한, 최근에는 Let's Encrypt 같이 무료(?) 인증서를 솔루션(?)으로 제공하는 방법도 존재하니 참고하자.  

## AWS에서 TLS 인증서를 적용할 3가지 방법
3가지 방법이나 되나?  
* ELB에서 처리 (가장 기본적)
* CloudFront에서 처리
* EC2에서 직접 처리

## ELB에서 처리
가장 기본적인 처리 방법이다. AWS가 푼돈을 벌기 위해 그런 것은 아니겠지만, EC2에 직접 붙이는 기능은 공식적(?)으로 지원하지 않는다. 여기에 대해 생각한 이유가 있었는데 뭐였더라...
아무튼 Amazon Certificate Manager를 통해 관리되고 있는 인증서를 ARN으로 연결하여 적용할 수 있다. 이때 ELB에 적용된 Security Group에서 443포트를 열어두지 않으면 안된다.
예전에는 Certificate Manager가 존재하지 않아 IAM에서 인증서를 관리하였는데, 정작 IAM에서는 등록된 인증서가 확인되지 않고, ELB의 target항목에서 불편하게 처리해야 했다. 지금도 그 잔재가 남아있다.(모든 ELB에서 사라지기 전까진 남아 있을 듯)
ELB에서 TLS 를 처리하고 나면, EC2까지는 80포트로 데이터가 전송되게 된다. 즉, EC2의 SG에는 ELB로 부터 들어오는 in-bound traffic에 대해 80포트를 열어둬야 한다.

## CloudFront에서 처리
공식적으로 (https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/RoutingToS3Bucket.html#routing-to-s3-bucket-prereqs) S3의 Static website hosting을 적용했을 경우 HTTP만 허용하게 된다. 이때 HTTPS를 적용하고 싶다면 어쩔 수 없이 CloudFront를 사용하여 TLS 인증서를 전처리 해야 한다.

## EC2에서 직접 처리
* EC2의 reverse proxy(NGinx)에 적용
  + Elastic Beanstalk를 사용하게되면 기본적으로 EC2에 reverse proxy를 적용하게 된다. 보안 때문인것으로 추측. 이 reverse proxy에서 TLS 인증서를 처리하게 하는 것이 간편하다.
* Tomcat같은 WAS에서 처리
  + reverse proxy가 없거나, 직접 처리하고 싶다면 Tomcat에서 직접 처리가능하다. Tomcat에서 TLS를 처리하는 방법은 여러곳에서 안내하고 있으니 참고
  
  