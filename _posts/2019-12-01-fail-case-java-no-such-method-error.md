---
layout: post
title:  "금주의 실패사례 - NoSuchMethodError의 정체"
date:   2019-12-01 17:00:00 +0900
categories: Java
comments: true
---
# NoSuchMethodError 해결
잘 동작하던 Spring Boot Web Application에서 실행 중 NoSuchMethodError가 발생하기 시작했다.

## 에러 발생 개요
NoSuchMethodError는 없는 메소드를 호출할 때 발생한다. 그런데 그렇다면 compile할 때 error가 나야 하는거 아닌가?  
실제로 error가 발생하는 java 파일을 살펴보면 없다고 하는 method가 잘 있다.
* 문제 사례
  + Spring에서는 error를 처리하는 방법을 다양하게 제공하고 있다. 본인은 org.springframework.boot.web.servlet.error.ErrorController를 구현(implements)하여 error 를 처리하고 있었다.
  + 며칠전까진 잘 됐는데, 어느날 갑자기 404 같은 error 처리시 아래와 같은 메시지를 띄우며 에러가 발생
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
* Git의 과거 revision을 checkout하여 실행
  + 과거 동작했던 revision도 안됨
* 밤 늦게까지 보고 또 보고 별 짓을 다해봄
  + 푹 쉬고 다음날 확인해보자 - 중요!

## 해결
* NoSuchMethodError를 다시 생각해봄
  + [정의](https://docs.oracle.com/javase/9/docs/api/java/lang/NoSuchMethodError.html)
    - Thrown if an application tries to call a specified method of a class (either static or instance), and that class no longer has a definition of that method.
    - Normally, this error is caught by the compiler; this error can only occur at run time if the definition of a class has incompatibly changed.
    - 즉, Runtime에서 발생하는 경우는 오직 class가 변경되었을 때 뿐
  + 로컬 개발환경에서 잘 존재하던 method가 runtime시 없다고 한다면, build 과정에서 의존성(dependency)이 꼬였을 것!
    - Compile할 때 참조하는 library와 실제 load(또는 link)시 참조하는 library가 다르다면?! 
    - build.gradle을 확인해보자
  + Spring Boot 버전 올림
    - 버전을 올리면, 의존성을 다시 받으면서 해결되지 않을까? - 여전히 안됨
  + Spring Boot starter 패키지에 포함되어 있지만 별도로 정의가 된 library들 제거
    - 본인의 경우 ApplicationHttpRequest가 참조하는 javax.servlet-api를 제거
    - *해결!*
    - 이게 왜 gradle에 들어있는거지?
* 원인 정리
  + 버전에 따라 class의 method가 다를 수 있음
    - 현재 버전의 Spring Boot의 starter 패키지에 포함되어 있는 javax.servlet.ServletRequest 와 별도로 특정한 javax.servlet-api 에 있는 javax.servlet.ServletRequest 가 달라서 method를 찾지 못했던 것
  + Gradle에서 cache하고 있는 library와 build시 사용되는 library가 다를 수 있음
    - 여러가지 중복된 library 의존성이 문제
  + 실제 여러 다른 사례에서도, library 버전이나 classpath 문제가 원인이었다고 함
    - 중복 class 존재: http://egloos.zum.com/slog2/v/3775114
    - 버전 정리: https://stackoverflow.com/questions/29444650/spring-boot-error-java-lang-nosuchmethoderror-org-apache-tomcat-util-scan-stan
    - Library 버전 정리 or Exclude: https://stackoverflow.com/questions/54122724/spring-boot-java-lang-nosuchmethoderror-javax-servlet-http-httpservletrequest
  
## 결론
* NoSuchMethodError는 의존성 문제
* 개발 시 마구잡이로 library를 프로젝트에 올리지 말자
  + 반드시 library간의 의존성을 확인하여 제외할 것은 제외하고 버전을 특정하여야 함
  + 특히 여러명이서 개발 할 때 주의 (자기한테 필요한 library를 마구 올림)