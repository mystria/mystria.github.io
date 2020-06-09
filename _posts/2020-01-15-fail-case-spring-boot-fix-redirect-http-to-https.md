---
layout: post
title: 금주의 실패사례 - Reverse Proxy 뒤의 Spring Boot가 http로 redirect되는 문제
date: 2020-01-15 18:00:00 +0900
categories: Spring Boot Java
comments: true
---
# Spring Boot로 개발한 웹앱(Web Application)에서 redirect를 하니 https가 풀리는 문제
보통 웹앱 개발 시 로컬환경에서 동작시켜 볼 때는 http 프로토콜로 localhost:8080을 쓰기 마련이다.  
그리고 당연하게도 인터넷에 배포할 때는 보안을 위해 https 프로토콜을 사용한다.  
그런데! https로 배포된 웹앱에서 발생하는 리다이렉션에서 https가 아닌 http 프로토콜로 보내버려 문제가 발생하였다.  
즉, Spring Boot 에서 redirect:나 sendRedirect()를 하면, https에서 http로 scheme이 바뀌는 경우가 생긴다.

## 해결
* 늘 그렇듯 StackOverflow 에서 확인
  + [Spring MVC “redirect:” prefix always redirects to http ? how do I make it stay on https?] (https://stackoverflow.com/questions/3401113/spring-mvc-redirect-prefix-always-redirects-to-http-how-do-i-make-it-stay)
  
* 다양한 이유가 있을 수 있겠지만,
  + WAS 자체는 https가 활성화 되어 있지 않기 때문에 자연스럽게 http를 이용하는 것으로 보임
    - 보통 Web Application Server(이하 WAS)가 Reverse Proxy Server 뒤에 위치하고 있을 때
    - 또는 SSL termination이 WAS가 아니라 그 앞단(LoadBalancer 등)에서 발생할 때

* Spring Boot에서 해결책
  + 접근한 client 정보를 활용하여 프로토콜을 유지하는 방법
  + application.properties 파일에 아래와 같이 설정을 주면 자동으로 인식하여 해결
    ~~~
    server.tomcat.remote-ip-header=x-forwarded-for
    server.tomcat.protocol-header=x-forwarded-proto
    ~~~
  + 그냥 Spring일 경우에도 해결책들이 다양하게 제시되고 있음(처음 말했던 StackOverflow 참조)

## 참고
* https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-enable-https
* https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Forwarded-For
* https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/X-Forwarded-Proto
* https://stackoverflow.com/questions/53911235/redirect-from-http-to-https-in-spring-boot-2-0-4





