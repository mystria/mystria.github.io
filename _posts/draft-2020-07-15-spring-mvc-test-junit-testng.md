---
layout: post
title:  "Spring MVC를 테스트하는 방법"
date:   2020-07-15 24:00:00 +0900
categories: Java Test Spring
comments: true
---

# Spring MVC를 테스트하는 방법
보통 테스트를 입문할 때는 아주 간단한 POJO 클래스를 만들고 한 눈에 이해되는 메소드를 테스트 한다. 메소드 하나에 파라메터 두 개를 넣고 돌려받은 리턴값을 비교...  
그러나 (나태한 변명이겠지만) 실무에 이를 적용하는 것은 쉽지 않았다. 수 십 개의 클래스들과 그 안에 꼬여있는 수 백 개의 메소드들.. 특히 Spring MVC를 개발하다보면, controller는 어떻게 테스트 하고, service는 어떻게 테스트할지, Spring Context는 어떻게 할지, 레거시 플랫폼으로 부터 받아오는 계속 바뀌는 데이터는 어떻게 고려해야 할지 등. 좀 막막하다.  

## Spring MVC의 context를 깔고 테스트
* Spring MVC는 Spring Boot처럼 혼자 뚝딱 실행되지 않는다. Tomcat같은 container가 필요한데... 테스트는 어떻게하지?
  + 다행히 spring-test 라이브러리가 standalone하게 Spring component들을 로드하여 실행시켜 줌
* Bean 들은 어떻게 주입하지?
  + @Autowired로 무책임하게 관리해오던 bean들은 그냥 테스트 코드를 실행하면 절로 만들어지지 않음
  + 테스트 시작과 함께 SpringContext를 불러와 필요한 곳에 주입해줘야 함

## Gradle.. 그리고 JUnit과 TestNG
* (특정 IDE에서만 일어날 수 있는 일이지만) Test 클래스를 그냥 실행하면 Gradle로 실행됨, 물론 Gradle로 테스트를 수행할 수도 있음
* Gradle로 실행하게되면 JVM parameter같은 것을 기존 방식으로 전달할 수 없음(뭔가 방법이 있는 듯)
* JUnit과 TestNG: 각각의 특징이 있음, 비교는 여기...
  + JUnit으로 테스트 하기
  + TestNG로 테스트 하기

## Controller와 그냥 Service(Component)테스트
* MockMVC
* 

## Config 읽어오기
* Class로 된 Config 파일
* XML로 된 Config 파일