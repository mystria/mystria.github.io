---
layout: post
title:  "금주의 실패사례 - REST API의 response가 자꾸 잘려서 오는 문제"
date:   2021-12-25 16:00:00 +0900
categories: JSON Java
comments: true
---
# 금주의 실패사례 - REST API의 response가 자꾸 잘려서 오는 문제

## 요약

서버의 JSON 응답이 반환되다가 끊김 → 메모리 문제, 혹은 디스크 문제로 추정

## 개요

개발용 REST API 서버와 통신하는 client 에서 자꾸 에러가 발생하여 서버 로그를 확인해 봤더니 아래와 같은 메시지가 있었다.

> com.fasterxml.jackson.databind.JsonMappingException: Unexpected end-of-input in VALUE_STRING
> 

인터넷에 검색해보면 [content-length가 문제](https://stackoverflow.com/a/36721932/8350542)라거나, [Tomcat문제](https://stackoverflow.com/a/60737964/8350542)라는 글이 있다. 하지만 전에도 비슷한 경험이 있어 느낌이 딱 온게 response 가 잘린 것 같았다. 

그래서 PostMan으로 해당 API server로 직접 request를 재연해보니 역시 response가 이상한 곳에서 딱 잘려오는 것을 확인할 수 있었다.

```json
// 제대로 올 때
{
  "name": "Kim",
  "age": 10,
  "address": "Seoul"
}

// 문제 상황
{
  "name": "Kim",
  "ag
```

## 원인 분석

### 현재 상황

- 대상 서버는 AWS ElasticBeanstalk 에서 동작하고 있으며, 오랫동안 별 문제 없이 잘 사용하고 있었음
- 문제가 되는 response 의 길이가 좀 컸기 때문에 용량 초과로 잘리는 것으로 추측
    - 짧은 응답이 오도록 변경하여 테스트 했더니 정상적으로 응답
    - 그런데 다시 길게했을 때, 1MB정도에서 잘림
    - 1MB 정도로 제약이 있는지 확인해 봤으나, 응답 크기는 크기 제한이 없음   [https://stackoverflow.com/a/8937620/8350542](https://stackoverflow.com/a/8937620/8350542)
- 그래서 timeout 문제인가도 생각해봤지만, 실제로 응답은 2초만에 실패 처리가 되었으며, API 서버의 네트워크 트래픽, CPU, response time 모두 정상

### 도대체 뭘까...

- 혹시 request 가 최적화 되지 않아서 그런가해서 다양하게 수정해보고, 응답으로 와야할 데이터도 좀 지워보고 별 짓을 다했지만 이유를 알 수 없음
- 계속 시도해보면 330KB에서 잘리거나 660KB에서 잘리거나 1MB에서 잘리거나... 그냥 아무 이유 없이 긴 response는 잘림
- 응? 짧은 response는 안 잘린다고?

### 혹시 메모리?

- 순간 머리속에 떠오른 network 와 buffer, bucket size 같은 오래된 학부 수업 (아련…)
- 얼른 API 서버로 들어가 메모리를 확인해보니 약 98% 사용 중
- JVM heap 확인해보니 30%정도로 설정되어있음, 나머진 메모리 누수?  
(참고로 AWS EC2 는 자체적인 memory 모니터링 기능이 없음)

```bash
[ec2-user@ip-10-0-0-999 ~]$ free
             total       used       free     shared    buffers     cached
Mem:       7953168    7433876     519292         72      88368     601248
-/+ buffers/cache:    3344260    4608908
Swap:            0          0          0
```

- 비교를 위해 동일한 war가 돌고있는 staging 환경의 서버를 확인해보니 메모리 50%사용 중

### 서버 재시작

지금까지 잘되다가 갑자기 안되고, 유사한 상황의 staging은 이상이 없고, 그럼 뭐다.  

서버 재시작!  

재시작 후 약간 버벅이기긴 했지만, 곧 정상응답을 확인했다. 참고로 에러 로그 전문은 아래와 같다.

```bash
Error occurred: org.glassfish.jersey.client.JerseyInvocation.translate(JerseyInvocation.java:856)
javax.ws.rs.client.ResponseProcessingException: com.fasterxml.jackson.databind.JsonMappingException: Unexpected end-of-input in VALUE_STRING
 at [Source: (org.glassfish.jersey.message.internal.ReaderInterceptorExecutor$UnCloseableInputStream); line: 1, column: 341422] (through reference chain: ..생략..)
	at org.glassfish.jersey.client.JerseyInvocation.translate(JerseyInvocation.java:856)
	at org.glassfish.jersey.client.JerseyInvocation.lambda$invoke$1(JerseyInvocation.java:750)
	at org.glassfish.jersey.internal.Errors.process(Errors.java:292)
	at org.glassfish.jersey.internal.Errors.process(Errors.java:274)
	at org.glassfish.jersey.internal.Errors.process(Errors.java:205)
	at org.glassfish.jersey.process.internal.RequestScope.runInScope(RequestScope.java:390)
	at org.glassfish.jersey.client.JerseyInvocation.invoke(JerseyInvocation.java:748)
	at org.glassfish.jersey.client.JerseyInvocation$Builder.method(JerseyInvocation.java:432)
	at org.glassfish.jersey.client.JerseyInvocation$Builder.post(JerseyInvocation.java:333)
	at com.your.project.list(Service.java:100)
Caused by: com.fasterxml.jackson.databind.JsonMappingException: Unexpected end-of-input in VALUE_STRING
 at [Source: (org.glassfish.jersey.message.internal.ReaderInterceptorExecutor$UnCloseableInputStream); line: 1, column: 341422] (through reference chain: ..생략..)
	at com.fasterxml.jackson.databind.JsonMappingException.wrapWithPath(JsonMappingException.java:394)
	at com.fasterxml.jackson.databind.JsonMappingException.wrapWithPath(JsonMappingException.java:353)
	at com.fasterxml.jackson.databind.deser.BeanDeserializerBase.wrapAndThrow(BeanDeserializerBase.java:1711)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:290)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:151)
	at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:286)
	at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:245)
	at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:27)
	at com.fasterxml.jackson.databind.deser.impl.FieldProperty.deserializeAndSet(FieldProperty.java:138)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:288)
Caused by: com.fasterxml.jackson.core.io.JsonEOFException: Unexpected end-of-input in VALUE_STRING
 at [Source: (org.glassfish.jersey.message.internal.ReaderInterceptorExecutor$UnCloseableInputStream); line: 1, column: 341442]
	at com.fasterxml.jackson.core.base.ParserMinimalBase._reportInvalidEOF(ParserMinimalBase.java:664)
	at com.fasterxml.jackson.core.base.ParserMinimalBase._reportInvalidEOF(ParserMinimalBase.java:641)
	at com.fasterxml.jackson.core.json.UTF8StreamJsonParser._loadMoreGuaranteed(UTF8StreamJsonParser.java:2373)
	at com.fasterxml.jackson.core.json.UTF8StreamJsonParser._finishString2(UTF8StreamJsonParser.java:2458)
	at com.fasterxml.jackson.core.json.UTF8StreamJsonParser._finishAndReturnString(UTF8StreamJsonParser.java:2438)
	at com.fasterxml.jackson.core.json.UTF8StreamJsonParser.getText(UTF8StreamJsonParser.java:294)
	at com.fasterxml.jackson.databind.deser.std.StringDeserializer.deserialize(StringDeserializer.java:35)
	at com.fasterxml.jackson.databind.deser.std.StringDeserializer.deserialize(StringDeserializer.java:10)
	at com.fasterxml.jackson.databind.deser.impl.FieldProperty.deserializeAndSet(FieldProperty.java:138)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:288)
```

## 문제 재발

### 상황 악화

- 그러나 문제가 또 발생하기 시작함!
- 게다가 이제는 서버가 자꾸 죽음
- Elastic Beanstalk에서 서버 로그를 100개 정도 조회해 보면 아무런 에러가 없음
- 그런데 Full logs를 당겨보면 실패, 왜 그렇지? S3에 저장되어있는 ElasticBeanstalk 로그를 보자...  
(참고로 EB는 자동으로 로그를 모아 S3에 저장해 준다.)

### 로그가 없음

없다. 없으면 안되는데...

- 문제가 발생한 것 같은 시점으로 거슬러 올라가 로그를 확인
- 로그가.... 5GB가 넘는다!
    - 그렇다, 우리 서버의 EBS는 32기가인데 매시간 로그가 5기가씩 쌓이는 것이다.
    - 참고로 EB는 로그를 6시간치 보관하고 있음
- 대량의 로그 → 끝없이 누수되는 메모리 → 서버는 간단한 기본 동작만 수행 중

### 조치

- 로그를 삭제하자!
- log4j2.xml 또는 logback.xml 을 이용하여 info level 이상만 찍게 설정
- debugMode를 false로 하여 불필요한 log 처리 방지
    - [https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=minjincodi&logNo=60152352](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=minjincodi&logNo=60152352)
    - log level 로 출력을 막더라도 log 로직은 수행됨
    - log 에 출력할 메시지를 생성하는 단계에서 메모리를 소모하지 않도록 방지

## 결론

문제의 1차적인 원인은 특정 배포 시점 이후 급증한 로그로 추측된다. 특정 로직에서 많은 반복과 긴 로그의 조합으로 디스크 공간을 많이 소진하였다.  
2차로 부족한 디스크 탓에 배출되지 않은 로그가 메모리에 쌓이며 메모리 부족을 발생시켰고, 결과적으로 길이가 긴 response 를 만들 여유조차 없어 응답을 잘라 버린 것으로 추측된다.

**응답이 잘린다면 서버의 디스크와 메모리를 확인해 보자.**  
Cloud 환경이라면 서버를 scale-up 또는 scale-out 하는 것으로 임시 조치가 가능할 것이고, 이 후 코드를 정비해야 할 것이다.