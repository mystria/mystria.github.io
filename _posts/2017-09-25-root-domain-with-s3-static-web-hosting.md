---
layout: post
title:  "Amazon Route53에서 Root domain을 www로 Redirection"
date:   2017-09-25 16:00:00 +0900
categories: AWS Route53 S3
comments: true
---
# Amazon Route53에서 Root domain 설정

## 목표
- Web Browser에서 example.com으로 접속 할 경우, www.example.com으로 redirection 시키기

## 진행 절차
0. 웹 서버 또는 웹 페이지 구성
  - EC2또는 Elastic Beanstalk 등 으로 웹 서버 구축
  - S3를 통해 static web hosting 사용 [Hosting a Static Website on Amazon S3][aws-website-hosting]
  - 해당 웹 서버에 Route53을 이용해 www.example.com 도메인 연결 (CNAME record 등을 활용)
1. S3에 도메인 명(example.com)으로 된 Bucket만들기
2. 해당 Bucket도 static website hosting 활성화(Bucket의 Properties에서 enable 가능)
3. Request Redirection을 선택하여 target을 www.example.com으로 설정
4. Route53에서 example.com의 A record를 S3의 EndPoint로 설정
  - IP 주소가 아닌 Alias로 선택
  - 보통 EndPoint는 s3-website-{region}.awsamazon.com 임 (Alias로 지정 가능)
5. 일정 TTL이 지난 후 테스트
  - example.com으로 접속 시 www.example.com으로 접속 됨을 확인

## 참고 사항
- Route53에서는 Alias를 통해 AWS의 여러 resource를 레코드에 연결 가능함
- Route53
  - Hosting Zone, Record의 개수마다 비용이 발생
  - 여기서 Record의 query별 비용도 발생 하지만,
  - Alias로 연결된 Record를 query할 때는 비용이 발생하지 않는다. [Route53 비용 정보][aws-route53-pricing]
- Root domain
  - [루트 도메인 CNAME 사용하기][root-domain-ref]를 참조
  - Zone Apex, Naked domain 등으로도 불리며 A record로 밖에 지정이 불가능
  - 실제로 Route53에서 A record 이 외 다른걸로 지정하려 할 경우 sub domain으로 설정이 됨
- A record
  - IP 주소로만 설정 가능
  - CNAME record는 URL 등을 사용 가능

- 즉, Root domain은 A record 밖에 연결이 되지 않지만,
- Alias를 이용하여 S3의 EndPoint를 A record에 연결하고,
- S3의 EndPoint가 www인 sub domain으로 redirection 하는 것

[aws-website-hosting]: http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html
[root-domain-ref]: http://seapy.com/2079
[aws-route53-pricing]: https://aws.amazon.com/route53/pricing/
