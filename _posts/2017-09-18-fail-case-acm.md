---
layout: post
title:  "금주의 AWS 실패사례 - ACM의 장단점"
date:   2017-09-18 16:00:00 +0900
categories: AWS Failure
comments: true
---
# 이번 주 AWS를 사용하면서 겪은 실패 사례

## 진행 작업
- 새로 구입한 도메인의 SSL인증서 구매
  - AWS Certificate Manager 장점
    - 무료
  - 등록방법
    - 도메인을 등록하면, 도메인의 주인에게 인증 메일이 전송 됨
    - 인증 메일의 링크를 클릭하면 해당 도메인의 SSL인증서를 제공
    - 전에는 N.Virginia만 지원하는 줄 알았는데, 현재는 모든 리전에서 지원 [리전별 지원표][aws-region-table]

  - Amazon CA가 브라우저에 기본 탑제되는걸까?
    - Amazon Root CA 가 있음
    - 또는 Starfield Services Root CA, Starfield Class2 CA

  - AWS Certificate Manager의 단점
    - 단! AWS안에서만 사용가능
    - SSL의 Private Key를 받을 수 없기 때문
    - 따라서 현재 지원하는 서비스는 아래 4가지 뿐
      - Elastic Load Balancer
      - Elastic Beanstalk(사실상 ELB?)
      - CloudFront
      - API Gateway
    - 한 리전에 등록한 것을 다른 리전에 복사하여 사용 불가
    - ACM [FAQs][aws-acm-faq] 참조

  - 전체 서비스를 AWS에서 사용하기 때문에 ACM을 쓰면 좋을 줄 알았는데, EC2에 직접 SSL인증서를 설치할 수 없어, 사용에 약간 제약이 발생함
  - 결국 다른 CA에서 새로 구매할 예정

[aws-region-table]: https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/
[aws-acm-faq]: https://aws.amazon.com/certificate-manager/faqs/
