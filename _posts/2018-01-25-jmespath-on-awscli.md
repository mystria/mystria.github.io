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
    + aws cli의 filter 를 이용하여 걸러낸 결과의 포맷을 변경
    ~~~ sh
      $ aws --profile test --region us-east-1 ec2 describe-vpcs --filters "Name=cidr, Values=10.5*.0.0/16" --query "Vpcs[*].CidrBlock"
      [
        "10.50.0.0/16",
        "10.51.0.0/16"
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

    + JMESPath를 이용해서 filter 적용
    ~~~ sh 
      $ aws --profile test route53 list-hosted-zones
      $ aws --profile test route53 list-resource-record-sets --hosted-zone-id /hostedzone/ABCDE --query "ResourceRecordSets[?Type=='A'].[Name,Type,ResourceRecords[*].*]"
      [
        [
          "test.example.com.",
          "A",
          null
        ]
      ]
    ~~~
    + 위 예제로 알 수 있듯이, --filter로 수 많은 목록을 유의미한 크기로 줄이고, --query로 필요한 값만 뽑아 내는 식으로 사용이 가능
  * 심화 예제 : Array 안에 또 Array가 있을 경우에는 다음과 같이 조건(Filter)을 붙여줄 수 있다.
    + In-bound IP Range에 0.0.0.0/0 를 포함하고 있는 Security Group 찾기
    + Object[?Object[?Object[?condition]]] 처럼 구현
    + 결과물을 json처럼 표시하려면 {}를 이용
    ~~~ sh
      $ aws --profile test --region us-east-1 ec2 describe-security-groups --query "SecurityGroups[?IpPermissions[?IpRanges[?CidrIp=='0.0.0.0/0']]].{GroupId: GroupId, IpPermissions: IpPermissions[].{FromPort: FromPort, CidrIp: IpRanges[].CidrIp}}"
      [
        {
          "GroupId": "sg-0000aaaa",
          "IpPermissions": [
            {
              "CidrIp": [
                  "0.0.0.0/0"
              ],
              "FromPort": 80
            }
          ]
        },
        {
          "GroupId": "sg-abcd1234",
          "IpPermissions": [
            {
              "CidrIp": [
                  "0.0.0.0/0"
              ],
              "FromPort": 80
            }
          ]
        }
      ]
    ~~~
  * 참고 : [공식 예제](http://jmespath.org/examples.html#filtering-and-selecting-nested-data) 를 찾아봐도 Nest Filter에 대해서 부족한 점이 있었음
    + Object[?Object.Object.Object == 'condition'] 과 같은 형태로는 조회 불가
