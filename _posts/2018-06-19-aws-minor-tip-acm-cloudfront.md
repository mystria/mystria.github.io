---
layout: post
title:  "AWS 미립자 팁 - CloudFront의 ACM 갱신"
date:   2018-06-19 16:00:00 +0900
categories: AWS ACM CloudFront
comments: true
---
# ACM과 CloudFront
ACM은 Amazon Certificate Manager, SSL 인증서를 관리해주며, ARN으로 AWS 서비스들에게 Certificate를 제공해준다.  
CloudFront는 CDN(Content Distribution Network)으로 자체적인 "cloudfront.net" domain으로 제공이 되지만, ACM을 이용해 custom domain을 적용 할 수도 있다.

## Certificate가 만료될 경우
  * ACM에서는 무료로 자체 Certificate가 발급 가능
    + 자체 Certificate는 자동(?)으로 리뉴얼이 됨
  * 외부 Certificate를 Import 가능
    + Import된 Certificate는 Reimport 기능이 있음
    + 이 Reimport로 만료된 Certificate를 갱신

## CloudFront로 배포된 domain일 경우
  * CDN은 일종의 cache
    + Cache 처럼 원격 Edge Location에 복제가 존재하고 있음
    + 이때 Certificate도 원격지에 복제로 존재
      - Origin(원래 소스위치)에서 ACM을 업데이트 하더라도, 이것이 원격지에 바로 반영되지 않음
      - 몇 시간(개인적으로 4시간)정도 지나면 업데이트 됨
  * 강제로 업데이트 하려면?
    + Reimport 하지말고 새로운 Certificate로 import한 후 CloudFront에서 해당 ARN으로 변경
    + CloudFront domain으로 변경 후 다시 custom domain을 재적용
    + 참고:[(StackOverflow)AWS Certificate Reimport reflection on CloudFront](https://stackoverflow.com/questions/46649171/aws-certificate-reimport-reflection-on-cloudfront)

## ELB에 적용된 Certificate
  * 예상과 같이 바로 갱신이 적용됨
