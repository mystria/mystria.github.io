---
layout: post
title:  "금주의 AWS 실패사례 - Resource가 삭제된 CloudFormation stack을 update하다 dead-lock"
date:   2018-03-08 17:00:00 +0900
categories: AWS CloudFormation
comments: true
---
# CloudFormation에서 Update stack을 했다가 멈춤
  UPDATE_ROLLBACK_IN_PROGRESS 상태에 빠진 경우 어떻게 해결할까?

## 발생 경과
* CloudFormation(이하 CF)의 stack으로 ElasticBeanstalk(이하 EB) environment를 생성(아마 다른 resource도 유사할 듯)
* 해당 EB environment를 수작업으로 삭제
* 해당 CF stack을 Update
* 당연히 없는 resource라서 실패 - Rollback 상태로 돌입
* 없는 resource라서 Rollback중 stuck : UPDATE_ROLLBACK_IN_PROGRESS

## 해결 시도
* Time out될 때 까지 대기 : 50분 정도로는 어림없음
* AWS CLI로 시도 : 불가능
~~~
An error occurred (ValidationError) when calling the DeleteStack operation: Stack [arn:aws:cloudformation:us-east-1:############:stack/your-stack-name-####/1sfeb4940-e373-11e7-95ee-50d5cad95262] cannot be deleted while in status UPDATE_ROLLBACK_IN_PROGRESS
~~~
* 삭제된 resource의 수동 재생성 : 50분째에 시도
> 원본 스택의 템플릿과 일치하도록 리소스를 수동으로 동기화한 다음 계속해서 업데이트를 롤백합니다. 예를 들어, AWS CloudFormation에서 롤백하려는 리소스를 수동으로 삭제한 경우 원본 스택과 이름 및 속성이 동일한 리소스를 수동으로 생성해야 합니다. - AWS FAQ

  + https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-update-rollback-failed
  + 이렇게 하면 일단 UPDATE_ROLLBACK_FAILED 상태로 돌아갈 수 있음
  + 이후 삭제하든 어쩌든 할 수 있음 - 해결!

## 그 밖의 해결책
* 영문 AWS UserGuide에는 FAQ가 존재
  + https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-nested-stacks-are-stuck
* Support Center에서 Ticket을 열어 stop 시켜달라고 요청
  + https://stackoverflow.com/a/42888134/8350542
  + Basic Support Plan 사용자는 마땅히 요청할 곳이 없음
* 1시간 정도 기다리면 time out으로 멈춘다고 하나, 안된다는 의견도 존재
  + https://forums.aws.amazon.com/thread.jspa?messageID=705856
  + 경험상 다른 종류의 IN_PROGRESS는 time out이 존재함
