---
layout: post
title:  "AWS 미립자 팁 - Cron expression"
date:   2021-03-20 10:00:00 +0900
categories: AWS Cron
comments: true
---

# AWS에서 쓰는 Cron과 다른데서 쓰는 Cron 표현식이 달라요
AWS CloudWatch Event의 Cron expression과 다른 곳의 Cron expression이 다르다?  
* AWS는 맨 앞의 seconds 필드가 없음
* AWS, Unix(6자리) 와 Quartz(7자리) 의 차이
* 즉, AWS의 표현식 앞에 0 을 추가로 붙여주면 동일해 짐
* 서비스 간 딜레이로 초단위 정확성은 제공하지 못한다고 설명함
* 참고로 UTC 기준으로 동작

## 참고
* Unix 형식은 Quartz와 다르게 '초'단위가 없다
  + https://stackoverflow.com/questions/23846743/different-types-of-cron-expression
* Quartz의 문서
  + http://www.quartz-scheduler.org/api/2.3.0/org/quartz/CronExpression.html
* AWS의 Cron 표현식
  + https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html