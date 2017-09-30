---
layout: post
title:  "금주의 AWS 실패사례 - MongoDB Atlas와 Elastic Beanstalk, 그리고 SES"
date:   2017-09-28 16:00:00 +0900
categories: AWS ElasticBeanstalk EC2 SES Failure
comments: true
---
# 이번 주 AWS를 사용하면서 겪은 실패 사례

## MongoDB Atlas와 Elastic Beanstalk
- MongoDB Atlas의 실시간 Migration을 실제로 적용해보고자 함
  - 예전에 정리했던대로 [Live migration]({% post_url 2017-07-11-mongodb-live-migration %}) 수행
  - 대신 Source DB가 legacy DB가 아닌, 다른 VPC에 존재하는 MongoDB instance
    - 즉, 이미 VPC 내부에 MongoDB를 사용중인 서비스가 내부 MongoDB를 끊고, 외부 Atlas를 VPC Peering해서 쓰고자 하는 상황
MongoDB Atlas와 VPC간 Peering을 하면, 서비스가 위치한 VPC의 Subnet에는 Route table설정을 해야 함([참고]({% post_url 2017-06-20-Config-MongoDB-Atlas-with-AWS %}))
  - 이때, Atlas VPC <-> Service VPC 사이 양방향 통신이 가능한 줄 알았으나 아님!
    - Service VPC가 Atlas VPC를 호출 할 수 있으나, 그 반대는 불가능
    - 관련하여 담당자에게 문의를 하려 했으나 아직 이용요금 지불을 못해서 보류 중
  - 이로 인해, VPC Peering을 체결하더라도 Atlas DB는 Source DB를 접근하지 못해 실시간 Migration 기능을 사용 불가
  - 결국 Source DB에 Elastic IP를 붙이고, Internet Gateway로 route를 하고 나서야 해결
- 위 작업을 하기 위해 다양한 방법으로 Subnet설정을 바꿔 시도를 하다가 문제 발생
  - 실수로 Private Subnet의 Route Table을 삭제함
    - 삭제된 사실을 잊고 있었음
  - 이 상태에서 Elastic Beanstalk(이하 EB)의 서비스가 제대로 동작하지 않음
    - route가 안되니 당연한 것…
    - Restart App Server(s) 시도 -> 복구되지 않음
    - Rebuild Environment 시도 -> Instance에 waiting이 걸린다며 Rebuild 실패
    - Rebuild가 실패하여 Environment의 상태가 Ready로 바뀌지 않아 Configuration 변경, Log 확인 등 어떤 작업도 불가
    - Terminate Environment 시도 -> 뭔가 의존성이 존재하여 삭제 실패
  - 해당 EB의 Environment는 사실 CloudFormation으로 생성 된 것
    - CloudFormation 에서 의존성이 존재할 경우 parent(?) Stack은 child(?)가 삭제될 때까지 삭제 안됨
    - CloudFormation 에서 정의한 어떤 Stack 때문에 Environment가 terminate되지 않아 CloudFormation의 Stack을 제거
    - Environment 종료 성공
  - 수작업으로 Environment 생성
    - 그러나 Instance 생성 단계에서 자꾸 waiting이 걸려 실패
    - 에러메시지
~~~
    The EC2 instances failed to communicate with AWS Elastic Beanstalk, either because of configuration problems with the VPC or a failed EC2 instance. Check your VPC configuration and try launching the environment again.
~~~

  - VPC와 Subnet 확인 중 Route Table이 누락된 것을 확인
    - Route Table 생성 후 Environment 생성 성공
    - 최신 Platform으로 Environment가 생성되면서 호환성 문제로 서비스가 정상동작 하지 않음
    - 해당 Environment를 종료하고 새로 생성을 시도
    - EB Console의 Web UI가 바뀐 뒤로 Environment의 Platform 버전을 바꾸는 곳을 찾을 수가 없음
    - AMI를 바꿔봐도 안됨
    - 한참을 헤매다 발견
      - 기존 UI : Environment Type 탭의 Platform 선택 근처에 바로 존재
      - 새로운 UI : Configure more options > Configuration presets 바로 하단에 존재
    - 해결
    - 쓸데 없는 걸로 하루를 낭비하여 이를 기록 함

# SES (Simple Email Service)
- SES는 AWS에서 메일을 발송할 수 있게 해주는 서비스
  - 등록된 Email 주소를 통해서만 메일 발송 가능
  - Email Addresses에서 주소를 등록하면 해당 Email주소로 발송이 가능
    - 해당 AWS API를 호출할 Programatic user도 생성
    - 위 User에게 SES를 사용가능한 Policy도 할당
    - 이 User의 AccessKey와 SecretKey로 Email 발송
  - 실패
    - [Email address is not verified][aws-ses-errors]
  - Sandbox안에서는 등록된 Email 주소로만 발송 가능
    - 실제로 Email Addresses에 있는 Send a Test Email 기능도 등록된 사용자에게만 가능
    - 테스트 할 때만 등록된 사용자에게 보낼 수 있는 줄 알았는데…
  - 계정을 만들면 기본적으로 SES는 Sandbox상태
  - Sandbox 해지 신청
    - [Moving Out of the Amazon SES Sandbox][aws-ses-sandbox]
    - 예전에 사용하던 계정에서도 Sandbox 해지 신청을 했었는데
    - 완전히 까먹고 있다가, 새로운 계정에서 계속 SES 메일 발송을 계속 실패하고 있었던 것
  - Working day 1일 후
  - 성공
- SES는 스팸메일을 양산할 수 있으므로 Amazon에서 엄격하게 reputation관리 중
  - SES에서 보낸 메일이 신고(constraints)처리 되거나 반송(bounced)되면 SES발송이 정지당함
  - Ticket을 열어 소명을 해야 다시 활성화 됨
  - Email Addresses에서 각 Email 주소별로 SNS Topic을 연결 할 수 있음
    - 이걸로 Constraint, Bounce, Delivery를 모니터링 가능
- SES에서 SMTP로 메일 발송 시
  - Email Sending > SMTP Settings 에서 설정 필요
  - Create My SMTP Credentials 를 통해 새로운 User를 만들고, 해당 User의 credential을 사용해야 함
    - 단순히 “Allow - ses:SendRawEmail” Policy를 가진 것 뿐인데
    - 임의로 생성한 User에게 해당 Policy를 주고, 이 User의 credential을 사용할 경우 SMTP 사용 불가
- 예전에는 안됐던거 같은데, 지금은 Email 수신도 가능
  - [Receiving Email with Amazon SES][aws-ses-receiving]
  - 단, 수신된 Email을 관리 할 순 없음
  - Email Receiving > Rule Set에서 수신한 메일을 S3에 저장하고 Lambda로 재전송하게끔 설정 가능

## 끝

[aws-ses-errors]: http://docs.aws.amazon.com/ses/latest/DeveloperGuide/ses-errors.html
[aws-ses-sandbox]: http://docs.aws.amazon.com/ses/latest/DeveloperGuide/request-production-access.html
[aws-ses-receiving]: http://docs.aws.amazon.com/ses/latest/DeveloperGuide/receiving-email.html
