---
layout: post
title:  "재정리 - Spring Boot Web 프로젝트에 XSS 적용하기"
date:   2020-04-20 24:00:00 +0900
categories: Java Spring Boot XSS
comments: true
---

# Spring Boot Web 프로젝트에 XSS 적용하기
작년 3월 메모처럼 갈겨 적었던 ["Spring Boot에서 json을 위한 XSS Filter 적용하기"](https://mystria.github.io/archivers/xss-filter-on-spring-boot)가 너무 무의미해 보여서 정리하고자 합니다.  
정리하는 겸 부족한 부분을 채우려고 했는데, 더 잘 정리된 글이 작성되었기에 소개합니다.  
    + [Spring Boot에서 JSON API에 XSS Filter 적용하기](https://jojoldu.tistory.com/470)

## Spring Boot Web 프로젝트
  * 본 글에서 말하는 Spring Web 프로젝트는 아래와 같음
    + __RESTful API를 지원__
    + Spring MVC에서 사용하는 애플리케이션
    + Embeded 된 Apache Tomcat container를 사용
  
## REST API
  * 많은 Web application 이 JavaScript 기반 front-end를 갖고, AJAX(XHR)를 통한 RESTful(REST) API로 back-end와 통신함
  * 이 REST API의 통신에 json이 request와 response에 활용됨
  * Request 와 response의 body에 XSS이 사용되지 않도록 escape가 필수

## XSS를 막을 2종류의 방법
  * 누군가 데이터에 아래와 같은 스크립트를 string으로 넣어두었을 때, 해당 값이 그대로 web page에 랜더링 되어버리면 문제가 발생
    ~~~ html 
    <script> alert('xss attack'); </script> 
    ~~~
  * 데이터에 escape처리를 하여 특수 문자를 아래와 같이 HTML Character Entity로 처리해줘야 함
    ~~~ html
    &lt;script&gt;alert('xss attack');&lt;/script&gt;
    ~~~
  * 그렇다면 스크립트를 어느 시점에서 escape해야 할까?
    + Request vs Response
    + [Http Message Converters with the Spring Framework](https://www.baeldung.com/spring-httpmessageconverter-rest)의 Client-Server Communication – JSON Only 항목 참고
  * Request(RequestBody, json)를 Java object로 바꿀 때 - __Deserialize__
    + [Spring Boot , Jackson Json Xss 처리 by @Circlee7](https://medium.com/@circlee7/spring-boot-jackson-json-xss-%EC%B2%98%EB%A6%AC-fdc85a18e9f2) 의 3-1 번 부분 참고
      - 별도의 deserializer class를 만들고
      - 대상 필드에 @JsonDeserialzer annotation을 통해 deserializer 적용하면
      - Request가 escape된 채로 Java object로 deserialize
  * Java object를 Response(ResponseBody, json)로 바꿀 때 - __Serialize__
    + [Spring Boot에서 JSON API에 XSS Filter 적용하기 by Jojolu](https://jojoldu.tistory.com/470) 의 결론 참고
      - 별도의 converter bean을 정의하여
      - MessageConverters에 등록되게끔 하면 
      - Response를 응답할 때마다 escape하여 json으로 serialize

## 선택은 시스템의 설계 및 요구사항에 맞추기
  * 각각의 장단점이 있겠지만, 개인적인 생각으로는
    + String에 스크립트를 허용하지않고, 해당 데이터가 다른 시스템에서도 읽힐 수 있으면 - Request를 escape
    + 스크립트도 일종의 콘텐츠이고, 처리하는 시스템들이 보증이 된다면 - Response를 escape
    + 아니면 escape character를 잘 정의하여 둘 다 escape!!

## MessageConverter로 Request를 escape하기?
  * 사실 이 방법을 찾기 위해서 그동안 고생했지만 아직 못찾음
  * ObjectMapper(jackson-databind)을 잘 튜닝하면?
  * 작성 중...

## 기타
  * ObjectMapper의 getFactory().setCharacterEscapes(new HTMLCharacterEscapes()); 부분의 getFactory()는 JsonFactory를 반환, 즉, serializer에서 쓰임
  * ObjectMapper의 ObjectWriter가 열쇠인것 같은데...


