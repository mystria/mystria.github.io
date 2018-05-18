---
layout: post
title:  "AWS CLI의 JMESPath"
date:   2018-01-25 18:00:00 +0900
categories: AWS CLI
comments: true
---
# JMESPath
AWS CLI(Command Line Interface)로 요청을 전달하면 json형식의 응답이 온다.  
애플리케이션 개발 시, 이 json을 여러 라이브러리로 처리할 수도 있지만, Shell 상에서 처리해야 할 경우에는 어떻게 해야 할까?  

## JMESPath란?
  * json 형식의 문서에 query를 할 수 있는 언어
    + json 문서에서 불필요한 내용은 지우고, 원하는 모양으로 재정의하기 좋음
    + 개인적으로 이 명세를 사용하는 곳은 AWS CLI 외엔 보지 못함
      - --query 라는 option 뒤에 따옴표를 적용하여 사용
  * 자세한 것은 역시 [Spec](http://jmespath.org/specification.html)을 참조하자

## Examples
  * Tutorial 좋음: http://jmespath.org/tutorial.html
  * AWS 상의 간단한 예제
  ~~~ sh
    $ aws --profile test --region us-east-1 ec2 describe-vpcs --filters "Name=cidr, Values=10.5*.0.0/16" --query "Vpcs[*].CidrBlock"
    [
      "10.50.0.0/16",
      "10.51.0.0/16"
    ]

    $ aws --profile test route53 list-hosted-zones
    $ aws --profile test route53 list-resource-record-sets --hosted-zone-id /hostedzone/ABCDE --query "ResourceRecordSets[?Type=='A'].[Name,Type,ResourceRecords[*].*]"
    [
      [
        "test.example.com.",
        "A",
        null
      ]
    ]

    $ aws --profile test --region us-east-1 ec2 describe-instances --filter "Name=network-interface.association.public-ip, Values=54.*" --query "Reservations[*].Instances[*].[InstanceId,PublicIpAddress]"
    [
      [
        [
          "i-xxxe0cv9s32d62334",
          "54.35.112.0"
        ]
      ],
      [
        [
          "i-xxx946856ea4xxxaa",
          "54.17.43.0"
        ]
      ]
    ]
  ~~~
  * 위 예제로 알 수 있듯이, --filter로 수 많은 목록을 유의미한 크기로 줄이고, --query로 필요한 값만 뽑아 내는 식으로 사용이 가능
