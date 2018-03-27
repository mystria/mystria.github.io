---
layout: post
title:  "3가지 종류의 S3 bucket policy"
date:   2018-03-26 17:00:00 +0900
categories: AWS S3
comments: true
---
# Amazon S3에서 IAM사용자(User 또는 Role)의 접근을 제어할 수 있는 bucket policy
S3에서 "특정한 사용자 외엔 접근불가"라는 규칙을 적용하고 싶다!

## 3가지 방법
사실 크게 2가지 방법으로 나눌 수 있다. 그러나 이러한 방법을 확인하기 위한 여러 시도들 중에 Condition을 이용한 방법은 2가지로 나눌 수 있었다. 총 3가지로 하여 정리하였다.  
* Principal을 이용한 접근 제어
가장 일반적인 방법, AWS에서도 Principal을 사용하길 권장  
  + Principal에 지정된 resource(User/Role)들에 대해 내부처리를 통해 고유ID를 찾아 권한 부여
  + 해당 User나 Role을 지우고 새로 만들게 되면 고유ID가 달라 인증이 안됨
  + Bucket policy를 재저장하면 대상의 고유ID가 갱신됨
  + 이름 변조를 통한 권한 편취를 막을 수 있음
    - UserName으로 허용되었을 경우 기존 IAM사용자를 삭제하고, 새로 사용자를 동일한 이름으로 만드는 방법 등의 꼼수
  + 단, wildcard를 지원하지 않음(이유는 상기 특징 때문)
  + Example
  ~~~ json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Deny",
                "NotPrincipal": {
                    "AWS": [
                        "arn:aws:iam::012345678910:root",
                        "arn:aws:sts::012345678910:role/{roleName}",
                        "arn:aws:iam::012345678910:user/{userName}",
                        "arn:aws:sts::012345678910:assumed-role/{roleName}/{userName}",
                        "arn:aws:iam::012345678910:role/{ec2ProfileName}",
                        "arn:aws:sts::012345678910:assumed-role/{ec2ProfileName}/{ec2InstanceId}"
                    ]
                },
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::{bucketName}"
                ]
            }
        ]
    }
  ~~~
  + Role도 허용하고, 해당 Role에 assume한 대상(User나 EC2 instance)도 허용해야함
  + 참고: [AmazonAWS Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#Principal_specifying)
* Condition을 이용한 접근 제어
유연하게 적용해야 할 경우 적절한 방법  
  + UserId(RoleId)를 이용한 접근 제어
  특정 사용자의 고유한 값을 이용하여 권한을 부여하므로 보안성이 좋으나, 해당 값을 알아내기 번거로움  
    - IAM User/Role/Group 등 IAM사용자 객체에는 고유ID가 존재
    - ARN도 아니고 "name"도 아니라 admin console에 노출 되지 않는 특별한 값임
    - 이 값을 알기 위해선 AWS CLI로 get-user/get-role/...를 해야함
    - 두 번째로 강한 보안, ID는 unique하기 때문에 이름 바꿔서 접속하는 것은 불가능
    - Wildcard를 통해 특정 Role을 사용하는 개체들에 대해 광범위한 허용 가능
    - Example
    ~~~ json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Deny",
                  "Principal": "*",
                  "Action": "s3:*",
                  "Resource": [
                      "arn:aws:s3:::{bucketName}"
                  ],
                  "Condition": {
                      "StringNotLike": {
                          "aws:userId": [
                              "ARO$$$$$$$$$$$$$$$$$$:{userName}",
                              "ARO$$$$$$$$$$$$$$$$$$:*",
                              "AID$$$$$$$$$$$$$$$$$$",
                              "012345678910"
                          ]
                      }
                  }
              }
          ]
      }
    ~~~
    - ARO는 Role, AID는 User의 접두어
  + ARN(UserName)을 이용한 접근 제어
  String으로 된 ARN이나 "name"요소를 이용하여 보다 완화된 접근을 제공  
    - Wildcard 지원
    - ARN 뿐 아니라 UserName과 같은 "name"에 대해서도 비슷하게 적용가능
    - ARN은 나름 고유하지만 "name"이 들어가 있기 때문에 유사한 "name"끼리 적용가능
    - 단, ARN이나 UserName은 악의를 가진 사람이 "name"을 바꿔서 권한을 편취할 수 있음
    - Example
    ~~~ json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Deny",
                  "Principal": "*",
                  "Action": "s3:*",
                  "Resource": [
                      "arn:aws:s3:::{bucketName}"
                  ],
                  "Condition": {
                      "ArnNotEquals": {
                          "aws:Arn": [
                              "arn:aws:sts::012345678910:assumed-role/{roleName}/{userName}",
                              "arn:aws:sts::012345678910:assumed-role/sample-role-name-*/*",
                              "arn:aws:iam::012345678910:user/{userName}",
                              "arn:aws:iam::012345678910:root"
                          ]
                      }
                  }
              }
          ]
      }
    ~~~
    - arn:aws:sts와 arn:aws:iam을 구분할 것
  + 참고: [AmazonAWS Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html)
