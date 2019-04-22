---
layout: post
title:  "금주의 AWS 실패사례 - AWS Java SDK로 Add Permission시 에러"
date:   2019-04-21 16:00:00 +0900
categories: AWS SNS SDK
comments: true
---
# AWS SNS에 별도의 permission을 추가하고자 함
Java Application 구현 시 AWS SNS(Simple Notification Service)를 생성하고, 여기에 Publish 할 수 있는 권한을 주기 위해 Java SDK를 사용하고자 하였다.  
그러나 SDK Document를 참고하여 구현한 코드는 의문의 InvalidParameter Exception을 반환할 뿐..

## Exception message
  * Policy statement action out of service scope!
    + InvalidParameter Exception
      ~~~ ssh
      com.amazonaws.services.sns.model.InvalidParameterException: Invalid parameter: Policy statement action out of service scope! (Service: AmazonSNS; Status Code: 400; Error Code: InvalidParameter; Request ID: ###)
      ~~~
    + Policy를 정의하는 문서의 action 이 서비스의 범위 밖이다 라고 이해하면 될 듯 함


## Document
  * AWS 공식 문서에는 SNS의 access control 관련 UseCase를 공유하고 있음
    + https://docs.aws.amazon.com/ko_kr/sns/latest/dg/AccessPolicyLanguage_UseCases_Sns.html
      ~~~ json
      {
		    "Version":"2012-10-17",
		    "Id":"AWSAccountTopicAccess",
		    "Statement" :[
          {
            "Sid":"give-1234-publish",
            "Effect":"Allow",           
            "Principal" :{
              "AWS":"111122223333"
            },
            "Action":["sns:Publish"],
            "Resource":"arn:aws:sns:us-east-1:444455556666:MyTopic"
          }
		    ]
		  }
      ~~~
    + Action 항목을 보면 "sns:Publish" 라고 되어 있음
  * AWS Java SDK 에서는 Amazon SNS action name 이 valid 하다고 함
    > Parameters:  
      actionNames - The action you want to allow for the specified principal(s).  
      Valid values: any Amazon SNS action name.
    + SNS action name이면 SNSActions model의 getActionName()이 생각남
    + com.amazonaws.auth.policy.actions.SNSActions
      ~~~ java
      /** Action for the Publish operation. */
      Publish("sns:Publish"),
      ~~~


## 관련 질문들...
  * 아무도 비슷한 증상을 겪지 않는 듯, 질문이 거의 없다.
    + https://forums.aws.amazon.com/thread.jspa?threadID=170302
    + AWS 포럼에 유사한 질문이 올라왔지만, 조금 상황이 다른 듯

## 해결책
  * 정답은 API Reference 에서 얻을 수 있었음
  * REST API call의 예제를 살펴보면
    ~~~ url
      https://sns.us-east-2.amazonaws.com/?Action=AddPermission
      &TopicArn=arn%3Aaws%3Asns%3Aus-east-2%3A123456789012%3AMy-Test
      &Label=NewPermission
      &ActionName.member.1=Publish
      &ActionName.member.2=GetTopicAttributes
      &AWSAccountId.member.1=987654321000
      &AWSAccountId.member.2=876543210000
      &Version=2010-03-31
      &AUTHPARAMS
    ~~~
    + ActionName을 member.N 이란 방식을 통해 여러개를 전달함
    + 근데 sns:Publish 가 아니네?
  * 간단하게 "Publish"로 작성하면 되는 것이었음

## 해결 코드
  ~~~ java
    // List<String> actionNameList = Arrays.asList(Publish.getActionName()); // not working
    List<String> actionNameList = Arrays.asList("Publish");
    AddPermissionRequest addPermissionRequest = new AddPermissionRequest(topicArn, label, awsAccountIdList, actionNameList);
  ~~~



