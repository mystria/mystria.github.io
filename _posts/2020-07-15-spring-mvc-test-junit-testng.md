---
layout: post
title:  "Spring MVC를 테스트하는 방법"
date:   2020-07-15 24:00:00 +0900
categories: Java Test Spring
comments: true
---

# Spring MVC를 어떻게 테스트 해야 할까?
보통 테스트를 입문할 때는 아주 간단한 POJO 클래스를 만들고 한 눈에 이해되는 메소드를 테스트 한다. 대충 메소드 하나에 파라메터 두어 개를 넣고 돌려받은 리턴값을 기대값과 비교...  
그러나 (나태한 변명이겠지만) 실무에 이를 적용하는 것은 쉽지 않았다. 수 십 개의 클래스들과 그 안에 꼬여있는 수 백 개의 메소드들.. 특히 Spring MVC를 개발하다보면, controller는 어떻게 테스트 하고, service는 어떻게 테스트 할지, Spring Context는 어떻게 할지, 레거시 플랫폼으로 부터 받아오는 계속 바뀌는 데이터는 어떻게 고려해야 할지 등. 좀 막막하다.  

## 참고
* 비슷한 고민을 한 좋은 정리 글
  + [How to Spring MVC Unit Test 스프링 MVC 단위 테스트](https://thswave.github.io/java/2015/03/02/spring-mvc-test.html)

## Spring MVC의 context를 깔고 테스트
* Spring MVC는 Spring Boot처럼 혼자 뚝딱 실행되지 않는다. Tomcat같은 container가 필요한데... 테스트는 어떻게하지?
  + 다행히 Spring MVC Test framework(spring-test 라이브러리)가 standalone하게 Spring context들을 로드하여 실행시켜 줌
    + [[Spring 레퍼런스] 10장 테스트 #2](https://blog.outsider.ne.kr/860)
    + [How to do in-container testing of a web app in Spring?](https://stackoverflow.com/a/20918796/8350542)
* Bean 들은 어떻게 주입하지?
  + @Autowired로 무책임(?)하게 관리해오던 bean들은 그냥 테스트 코드만 덜컥 실행하면 절로 만들어지지 않음
  + 테스트 시작과 함께 Spring context를 불러와야 bean들이 생성됨
    + JUnit으로 테스트 하기
      ~~~ java
      // Runner 클래스로 Spring환경으로 실행
      @RunWith(SpringJUnit4ClassRunner.class)
      @ContextConfiguration(locations= { "classpath*:applicationContext.xml" })
      public class Tests {}
      ~~~
    + TestNG로 테스트 하기
      ~~~ java
      // AbstractTestNGSpringContextTests 를 상속
      @ContextConfiguration(classes= { ServiceConfig.class })
      public class Tests extends AbstractTestNGSpringContextTests {}
      ~~~
  + ContextConfiguration으로 bean이 정의/설정되는 클래스나 XML 스팩들을 지정할 수 있음

## Controller와 그냥 Service(Component)테스트
* Controller 테스트 
  + Spring MVC에서 Controller는 어쩌면 client와 만나는 end point로 볼 수 있음
  + HTTP request의 header나 parameter를 전달하여 실제 HTTP로 호출하는 것 처럼 테스트
  + WebApplicationContext와 MockMVC를 통해 가상의 서버에서 동작하는것 처럼 동작
    + [Spring test와 Junit4를 이용한 테스트](https://effectivesquid.tistory.com/entry/Spring-test-%EC%99%80-Junit4%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%85%8C%EC%8A%A4%ED%8A%B8)
    + [spring-test - 테스트 프로젝트 및 테스트 케이스 기본구조](https://offbyone.tistory.com/290)
    + [Spring MockMvc(spring-test)](https://jdm.kr/blog/165)
      ~~~ java
      @RunWith(SpringJUnit4ClassRunner.class)
      @ContextConfiguration(locations= { "classpath*:applicationContext.xml" })
      // WebApp 어노테이션 필요
      @WebAppConfiguration
      public class CategoryControllerTest {
        @Autowired
        private WebApplicationContext webApplicationContext;

        private MockMvc mockMvc;
      }
      ~~~
  + 주입되는 service는 Mock으로 만들어 controller의 비즈니스 로직만 테스트도 가능 
* Service 테스트
  + @Service 어노테이션 등으로 정의된 Service bean은 context를 바탕으로 테스트 실행했기 때문에 언제든 주입가능
  + Spring 프로젝트 개발 하듯이 클래스와 메소드 생성하고 호출하여 테스트
    + [@ContextConfiguration Example in Spring Test](https://www.concretepage.com/spring-5/contextconfiguration-example-spring-test)
    + [Junit을 사용하여 단위테스트(Spring-Test사용)](https://devjjo.tistory.com/29)

## 정리
* Spring test framework를 이용해 context를 로드한 상태로 테스트 수행
* Controller는 MockMVC로 테스트
* Service나 다른 bean들은 평범하게 테스트
* Mock을 이용해 가상의 의존성을 주입하여 테스트 클래스의 로직만 테스트

## 기타: Gradle로 test 실행
* (특정 IDE에서만 일어날 수 있는 일이지만) Test 클래스를 그냥 실행(Run)하면 Gradle로 실행됨, 물론 Gradle로 테스트를 수행할 수도 있음
* Gradle로 실행하게되면 JVM parameter같은 것을 기존 방식으로 전달할 수 없음(뭔가 다른 방법이 있는 듯)
  