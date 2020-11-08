---
layout: post
title:  "AWS SES로 이메일 재전송하기 (부제: 고객 응대 용 메일 설정)"
date:   2020-11-09 24:00:00 +0900
categories: AWS SES
comments: true
---

# SES로 고객 응대 용 이메일을 CS 담당자에게 재전송 하자
AWS의 SES는 Email을 전송할 수 있는 (유료)서비스이다. 그냥 막 보낼 수 있는 것은 아니고, 인증된 메일 주소나 도메인으로만 발송할 수 있고, 스팸 메일 발송을 막기 위해 처음에는 등록된 주소로만(SandBox) 메일을 보낼 수 있다. AWS Support로 요청을하면 불특정 다수에게 메일을 보낼 수 있게 허락을 해준다. 하지만 reputation(평판)이라는 개념이 있어서 반송이 많이 되거나 수신인이 신고를 많이하면 발송이 차단될 수 있다. 내가 내 돈 주고 쓰겠다는 서비스인데 꽤나 까다롭다.  
어쨋든 이 서비스는 별도의 메일 서버를 구축할 필요없이 간편하게 AWS API로만 메일을 보낼 수 있기 때문에 개발자들에게 매우 유용하다. 그리고 메일 수신 기능도 제공하고 있기 때문에 Email을 통한 다양한 형태의 서비스를 구현할 수 있다.  
예를 들면, 고객 서비스용 메일이다. 고객의 문의나 요청등을 받을 수 있는 Email 창구가 하나씩 있고 이 Email을 CS 담당자가 받게 하는 시스템 같은 것이다. 이러한 시스템을 간단하게 구현하려면 Google G-Suite 같은 서비스를 구독하여 MX 주소를 Email주소에 연결해 두는 방법을 쓸 수 있다. 그러나 이미 다른 회사에서 쓰는 메일 서비스가 있는데, 이 웹사이트는 회사 메일 서비스와 도메인이 다르다면? 회사 메일 서비스를 내가 마음대로 설정할 수 없는거라면? 웹사이트 별로 다른 주소의 CS 메일을 한 곳에 몰아서 받아야 한다면?  
여러 복잡한 사정들로 인해 그냥 (간단하게?) 메일을 수신하여 다른 주소로 재전송 해야 하는 상황이 올 수 있다. AWS는 이러한 요구사항을 해결할 방법을 SES의 메일 수신/발신 기능을 통해 제공해주고 있다. 다소 번거로운 과정이 필요하지만...  

## 개요
AWS의 SES(Simple Email Service)에 2015년 도입된 기능  
https://aws.amazon.com/ko/blogs/korea/new-receive-and-process-incoming-email-with-amazon-ses/  
https://aws.amazon.com/ko/blogs/messaging-and-targeting/forward-incoming-email-to-an-external-destination/  
https://docs.aws.amazon.com/ko_kr/ses/latest/DeveloperGuide/receiving-email-getting-started.html  
설정대로 따라하면 쉬움  
그러나 간단히 요약해서 정리하자면,

## 작업 절차
0. 재전송 받을 이메일 준비
 - G-Mail 같이 다양한 filter와 forward를 활용할 수 있는 이메일 서비스가 적합
 - 중요한 분배 기능은 여기에 설정할 것

1. 도메인의 주인이어야 함, 도메인의 MX주소를 AWS SMTP서버로 설정
 - SES에 도메인을 등록/인증해야 함
 - Route 53에 도메인이 등록되어 있다면, SES에 도메인을 등록하면서 도메인 인증과 MX주소를 one-click으로 설정 가능
 - 보내는(재전송) 메일을 위한 이메일 주소도 하나 인증 필요

2. S3 Bucket을 만들어 받은 메일을 octet-stream으로 저장, 메일을 받아서 바로 넘길 수 없기에 임시 저장 필요
 - SES가 S3 Bucket에 파일을 등록할 수 있게 권한 부여 필요
 - 재전송하고 나면 메일 파일이 필요 없으므로 life-cycle을 설정해도 좋음

3. Lambda를 이용해 메일 재전송
 - Lambda가 S3에 접근하고, 로깅을 하며, 메일을 보낼 수 있도록 policy를 생성
 - 주의: SES는 검증된 특정 주소로만 메일 발신 가능 - 즉, 보낸 사람의 메일 주소를 발신자로 하는 식의 재전송은 안됨(그러나 그런것 처럼 보이게는 가능, 아래 오픈소스 참고)
 - AWS의 sample용 python코드는 아주 원시적인 코드라서 그냥 메일을 eml파일로 첨부하여 재전송함
 - 다른 사람의 node.js소스 활용 
   - https://github.com/arithmetric/aws-lambda-ses-forwarder
   - 완성도 높은 구현
   - 보낸 사람은 어쩔 수 없이 특정한 주소만 되지만 reply 값을 조작하여 원래 발신자로 표시 가능 
   - 필요에 따라 수신자를 다양하게 설정 가능
     - 수신된 주소에 따라 재전송할 주소를 다양화
     - 한 번에 여러 명에게 보낼 수도 있음
   - 설명서 https://techpolymath.com/2016/07/20/serverless-replacement-for-basic-email-services/

4. SES의 Rule Set설정
 - 여러 Rule Set을 만들 수 있지만 Region 별 Active Rule Set은 단 하나 (주의)
 - Rule Set안에 여러 rule 설정(priority 존재)
  1. S3에 저장
  2. Lambda를 이용해 재전송 (보통 event타입으로 trigger)
  3. Rule을 명시적으로 여기서 끝내도 되고, 안끝내면 다음 rule로 넘어감
  4. SNS 설정 등을 하여 다양하게 활용 가능
 - 여러 도메인의 메일을 각각 rule로 설정하여 한 Rule Set에서 처리

5. 요약
 - SES의 수신 기능으로 메일 수신
 - SES에서 S3에 이메일 저장하고, Lambda호출
 - Lambda가 S3에서 메일 파일을 읽어와 분석하여, 지정한 주소로 재전송
 - Gmail같은 메일 서비스에서 고급 룰 활용
 - Lambda와 S3 사용료 주의(메일 양이 많으면 그냥 다른 유료 서비스 이용)
 