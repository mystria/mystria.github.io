---
layout: post
title:  "금주의 AWS 실패사례 - .ebextensions로 S3를 접근할 instance profile의 인증 문제"
date:   2018-04-01 21:00:00 +0900
categories: AWS ElasticBeanstalk S3 IAM
comments: true
---
# IAM Role과 EC2 Instance Profile의 차이로 인한 문제
EC2에서 IAM Role의 권한들을 얻기 위해서는 Instance Profile이라는 container가 필요하다.  
그리고 ElasticBeanstalk에서 .ebextensions를 이용하여 EC2가 몇가지 file들을 S3로 부터 가져올 때, 이 Instance Profile을 필요로 하게 된다.  
AWS Docs에서는 이를 위한 예제를 제공하고 있지만, 그게 되지 않아 사흘간 고생한 사례이다.  
어찌보면 CloudFormation을 이용해 ElasticBeanstalk Application Environment를 생성한게 문제였을지도...

## 사건의 발단
ElasticBeanstalk를 이용하여 Amazon Linux서버를 띄우는데, 이 EC2에서 SSL인증서를 이용해 HTTPS 프로토콜을 처리하려고 한다.  
아시다시피 Elastic Load Balancer(ELB)에서는 자체적인 기능 지원을 통해 SSL 인증서를 적용할 수 있고, ELB와 EC2간의 통신은 HTTP로 처리한다. 그러나, 약간 더 높은 보안을 위해 ELB는 HTTPS를 통과 시키고 EC2 단에서 HTTPS통신을 처리하고자 한다면, 수고로움이 필요하다.  
  * EC2 에서 HTTPS 프로토콜 처리 방법
    1. 대상 instance에 SSL 인증서 파일을 넣는다.
    2. 대상 instance의 proxy에서(보통 apache 또는 nginx사용) 설치된 인증서를 이용하여 HTTPS 프로토콜을 처리한다.
    3. 참조: Load Balancer가 없는 Single Instance일때 HTTPS처리 방법 [Amazon AWS Docs](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/https-singleinstance.html)
  * ElasticBeanstalk(EB)에 적용
    + 위 방법을 EB에서 처리하기 위해서는 .ebextensions라는 설정 파일들을 이용해야 함
      - files를 이용하여 인증서 파일을 획득
      - nginx proxy에서 해당 인증서를 활용하게끔 설정: 참고 [nginx 설정]({% post_url 2018-03-16-fail-case-timeout-free-on-aws %})
      - Security Group및 ELB options 설정: 자세한 설명 생략
    + 그런데 files를 통해 S3의 파일을 가져오려면?
      - 해당 EC2의 Instance Profile을 이용하여 EC2에 권한을 제공하고, S3에 접근하여 파일을 획득
      - 이때 authentication 옵션이 Instance Profile을 지목해야함
      - 참고: files [Amazon AWS Docs](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-files)
      - 참고: 인증방법 [Amazon AWS Docs](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-resource-authentication.html)
  * 예제도 멀쩡히 제공되고 있지만, 안됨!

## 문제 분석
"Deploying new version to instance(s)." 단계에서 .ebextensions가 호출 되면서 stuck이 발생.  
결국 다음과 같은 404 오류를 반환하며 실패한다.
  * EB Environment events
  ~~~
    [Instance: i-00000abcd1234eeee] Command failed on instance. Return code: 1 Output: Failed to retrieve https://your-certs.s3.amazonaws.com/client.pem: [Errno 404] HTTP Error 404 : <?xml version="1.0" encoding="iso-8859-1"?>. EBExtension failed. For more detail, check /var/log/eb-activity.log using console or EB CLI.
  ~~~
  * 실제 문제를 일으키는 소스 코드
  ~~~ yaml
    files:
      /etc/pki/tls/certs/client.pem:
        mode: "000400"
        owner: root
        group: root
        source: https://your-certs.s3.amazonaws.com/client.pem
        authentication: S3Auth

    Resources:
      AWSEBAutoScalingGroup:
        Metadata:
          AWS::CloudFormation::Authentication:
            S3Auth:
              type: S3
              buckets: your-certs
              roleName:
                "Fn::GetOptionSetting":
                  Namespace: "aws:autoscaling:launchconfiguration"
                  OptionName: "IamInstanceProfile"
                  DefaultValue: "aws-elasticbeanstalk-ec2-role"
  ~~~
  * 원인을 추측
    1. YML파일로 되어있는(CloudFormation 코드) ebextensions파일의 따옴표 문제?
      - 실제로 공식 예제에는 위 코드와 달리 "S3"라든가 "S3Auth" 부분에 따옴표가 들어가 있음
    2. S3 Bucket의 bucket policy의 잘못된 설정?
      - SSL인증서는 아무나 가지면 안되므로 특정 IAM Role에 대해서만 허용하는 bucket policy가 적용 중
    3. source의 잘못된 URL?
      - 근래 https://BUCKET.s3.amazonaws.com이 아니라 https://s3.amazonaws.com/BUCKET 으로 바뀜
    4. EC2의 Security Group설정, Subnet의 Route table설정 등 네트워크 문제?

  * 위를 다 확인 했지만 아무 문제 없음
  * 남은 문제는 roleName

## 수상한 roleName
  * files의 인증(authentication)을 위해 roleName을 획득하는 부분이 존재
  * roleName에 대한 [공식 설명](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-authentication.html)
    + Describes the role for role-based authentication.
      - Important: The EC2 instance must be able to *access this role using an instance profile*.
    + EC2 instance는 Instance Profile로 IAM Role에 접근할 수 있음
  * roleName에 IAM Role name을 수동으로 입력해 보니, 원하던대로 S3에서 인증서를 가져옴
  ~~~ yaml
    S3Access:
      type: S3
      buckets: your-certs
      roleName: your-role
  ~~~
  * 그러나, Fn::GetOptionSetting으로 가져오는 것은 IamInstanceProfile
    + Instance Profile은 IAM Role을 EC2에 적용하기 위한 container
      - 참조: [Amazon AWS EC2 Docs](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
    + AWS CLI로 대상 ElasticBeanstalk의 설정값을 불러보면 Instance Profile을 확인 할 수 있음
    ~~~
      aws elasticbeanstalk --region us-east-1 describe-configuration-settings --application-name APP_NAME --environment-name ENV_NAME
    ~~~
  * roleName에 Instance Profile을 입력해 보니, 실패
  ~~~ yaml
    S3Access:
      type: S3
      buckets: your-certs
      roleName: your-instance-profile
  ~~~

## 원인은 EC2 Instance Profile
  * 보통 수동으로 EC2에 적용 가능한(Trusted Entity가 ec2.amazonaws.com) Role을 만들면, Instance Profile도 자동으로 생성 됨
    + 이때 Role Name과 Instance Profile은 자동으로 같은 이름으로 생성됨
      - Console을 이용할 경우
    + 물론, 다르게 생성할 수도 있음
      - AWS CLI를 이용할 경우
    + 참고: [Amazon AWS IAM Docs](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
  * 예상 동작
    + Instance Profile을 부르면 어떤 내부적인 프로세스를 통해 해당 IAM Role을 찾고,
    + IAM Role의 권한을 통해 S3 접근
  * 실제 동작(원인을 알 순 없지만)
    + 보통은 Instance Profile과 IAM Role의 이름이 동일하니깐,
    + Instance Profile이름으로 IAM Role을 찾고,
    + IAM Role의 권한을 통해 S3 접근 - 실패!!
  * 이 부분은 내가 AWS 내부 코드를 꿰고 있는 것이 아니므로 확신 할 수 없음, 단순한 나의 이해 부족일 수 있음!

## 해결책
  * IAM Role과 Instance Profile의 이름을 동일하게
    + ElasticBeanstalk를 통해 Environment를 생성할 때 IAM Role을 생성할 것인지 입력 가능
    + 이때, 이미 (Console로)만들어져 있는 IAM Role을 쓰면, 아무 문제 없음
    + IAM Role을 새로 만들어야 해도 Console일 경우 동일한 이름으로 Instance Profile을 생성하는 듯
    + 단, CloudFormation으로 생성한다면,
      - AWS::IAM::role을 생성할 때, RoleName속성을 입력할 것. 필수가 아니므로 지정하지 않으면 난수값을 넣어 ~Role이란 이름이 됨
      - AWS::IAM::InstanceProfile을 생성할 때, InstanceProfileName속성을 입력할 것. 필수가 아니므로 지정하지 않으면 난수값을 넣어 ~InstanceProfile이란 이름이 됨
    + 위 두 이름을 parameter로 받아 동일하게 하면,
      - Fn::GetOptionSetting으로 획득하는 IamInstanceProfile값으로 IAM Role을 획득 할 수 있음

## 후기
  * IAM Role이름과 Instance Profile의 이름이 매우 흡사하여 미처 눈치 못 챘던 것이 장시간 삽질을 하게 된 원인
  * 앞으로 자동 생성 이름을 신뢰하지 말자
