---
layout: post
title:  "금주의 실패사례 - NoSuchMethodError의 정체"
date:   2019-12-01 17:00:00 +0900
categories: Java
comments: true
---
# NoSuchMethodError 해결
잘 동작하던 Spring Boot Web Application에서 NoSuchMethodError가 발생하기 시작했다.

## 에러 발생 개요
NoSuchMethodError는 없는 메소드를 호출할 때 발생한다. 그런데 그렇다면 compile할 때 error가 나야 하는거 아닌가?  
실제로 error가 발생하는 java 파일을 살펴보면 없다고 하는 method가 잘 있다.
* 문제 사례
  + Spring에서는 error를 처리하는 방법을 다양하게 제공하고 있다. 본인은 org.springframework.boot.web.servlet.error.ErrorController를 구현하여 error 를 처리한다.
  + 며칠전까진 잘 됐는데 어느날 갑자기 error가 발생하자 아래와 같은 메시지를 띄우며 에러가 발생
    ~~~ ssh
    2019-12-01_18:00:00.000 ERROR [localhost] - Exception Processing ErrorPage[errorCode=0, location=/error]
    java.lang.NoSuchMethodError: javax.servlet.http.HttpServletRequest.getHttpServletMapping()Ljavax/servlet/http/HttpServletMapping;
      at org.apache.catalina.core.ApplicationHttpRequest.setRequest(ApplicationHttpRequest.java:708)
      at org.apache.catalina.core.ApplicationHttpRequest.<init>(ApplicationHttpRequest.java:114)
      at org.apache.catalina.core.ApplicationDispatcher.wrapRequest(ApplicationDispatcher.java:917)
      at org.apache.catalina.core.ApplicationDispatcher.doForward(ApplicationDispatcher.java:358)
      at org.apache.catalina.core.ApplicationDispatcher.forward(ApplicationDispatcher.java:312)
      at org.apache.catalina.core.StandardHostValve.custom(StandardHostValve.java:394)
      at org.apache.catalina.core.StandardHostValve.status(StandardHostValve.java:253)
      at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:175)
      at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
      at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
    ~~~

## 원인 
* 일단 갑자기 안되게 되었으니 바꾼 부분을 살펴봄
  + Error 처리를 하는 ErrorController 구현체의 수정을 모두 원복해 봤으나 안됨
* Git의 과거 revision을 checkout하여 확인
  + 과거 동작했던 revision도 안됨
* 늦게까지 보고 또 보고 별 짓을 다해봄
  + 푹 쉬고 다음날 확인해보자 - 중요!

## 해결
* NoSuchMethodError를 다시 생각해봄
  + 로컬 개발환경에서 잘 보이는 method가 없다고 한다면, 의존성이 꼬였을 것!
  + Spring Boot 버전 올림
    - 여전히 안됨
  + Spring Boot starter 패키지에 포함되어 있지만 별도로 정의가 된 library 제거
    - 본인의 경우 ApplicationHttpRequest가 참조하는 javax.servlet-api를 제거
    - *해결!*
* 원인 정리
  + 버전에 따라 Class의 method가 다름
    - 현재 버전의 Spring Boot의 starter 패키지에 포함되어 있는 javax.servlet.ServletRequest 와 별도로 특정한 javax.servlet-api 에 있는 javax.servlet.ServletRequest가 달라서 method를 찾지 못한 것


  
## 결론
* 간단한 문제이지만 마음이 쫓기다보면 눈이 흐려짐
* NoSuchMethodError는 의존성 문제
* 개발 시 마구잡이로 library를 프로젝트에 올리지 말자
  + 반드시 libarary간의 의존성을 확인하여 제외할 것은 제외하고 버전을 특정하여야 함