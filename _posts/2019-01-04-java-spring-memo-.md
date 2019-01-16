---
layout: post
title:  "Java 공부 - Spring Bean의 lifecycle(scope)"
date:   2019-01-04 03:00:00 +0900
categories: Java Spring
comments: true
---

# Spring의 세계
Spring을 이용해 코딩을 하다보니, 문득 각 bean들의 life cycle(혹은 scope)이 궁금해 졌다.  
오래 전에 Spring을 1주일 과정으로 짧게 배운적은 있었는데, 거의 다 까먹고 야매로 그냥 꾸역꾸역 동작하게만 만들며 살아온 그리 길지 않은 개발 인생이었다.  
과제를 새로 할 때 마다 지난 과제의 코드들이 부족해 보여서 속상하던 찰나, 어떻게하면 더 효율적인 코드를 구현할 수 있을지 의문이 들기 시작했다.  
의식의 흐름대로 조사하고, 답을 찾고 다시 의문을 가지며 글을 작성한다.  
부족한 내용에 부족한 글이다.  

## Spring Reference
* 모든 의문의 답은 레퍼런스에 있다.
  + https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/beans.html
  + https://docs.spring.io/spring/docs/3.0.0.M3/reference/html/ch04s04.html)
* 그러나 영어가 부족할 뿐...

## Bean의 Scope
* controller는 몇개가 만들어질까?
  + 정답은 singleton, bean별로 @Scope를 통해 조절 할 수는 있지만, 보통은 singleton
  + https://stackoverflow.com/questions/5667727/spring-controller-lifecycle
  + Spring의 기본 scope는 singleton임 (그래서 Thread-safe 구현이 필요)
  + 레퍼런스에 의하면 Spring singleton은 Java singleton 패턴과 약간 다르다고 함

* Spring singleton과 Java singleton의 차이?
  + Singleton의 기준이 각각 Container(또는 ApplicationContext)와 Classloader로 다름
  + 참조 : http://blog.daum.net/rollin/8097082
  + 간단히 말하면 Java singleton이 좀 더 물리적(?)인 범위에서 singleton이란 뜻

* 그러면 Spring MVC에서 여러 사용자(session 또는 request)가 동시에 접속했을 때 bean들이 singleton이라도 서로 간섭이 없나?
  + 없음, 각 method는 thread별로 실행이 되고, 이때 전달되는 parameter는 stack간 상호 연관이 없다.
  + https://stackoverflow.com/questions/17235794/how-does-spring-mvc-handle-multiple-users
  + https://stackoverflow.com/questions/19772437/spring-mvc-singleton-controller-multiple-download-requests
  + 그래서 thread-safe가 중요함
  + 물론, singleton scope가 아닌 경우에는 상황이 다르니 주의

* Singleton말고 다른 것도 있나?
  + 여러가지가 있음
    - singleton, prototype, request, session, global-session, application, websocket ... 
  + Scope 별 차이는 아래 사이트들 참고
    - http://javawebtutor.com/articles/spring/spring-bean-scopes.php
    - https://www.baeldung.com/spring-bean-scopes

* Singleton controller에 autowired로 주입(inject)된 bean들의 lifecycle은 어떨까? 당연히 singleton일까?
  + 정답은 그렇다, 그러나 다른 scope로 정의 가능
  + Spring reference에 따르면 bean이 주입될 때 scope가 적용된다고 함
  + Singleton은 딱 한번 생성되므로 주입도 딱 한 번 일어나고, prototype을 주입하더라도 이는 딱 한 번만 주입되므로 사실상 singleton과 수명을 함께함

* 이에 대한 해결책
  + 아래 링크에서 여러가지 해결책 제공
  + https://www.baeldung.com/spring-inject-prototype-bean-into-singleton
  + 이렇게까지 해야하는 경우가 있을까?

* Service layer의 경우에는 stateful한 경우(class의 전역변수?)가 많은데?
  + 확인해보니 session scope로 하는 것도 reasonable 하다 함
  + https://stackoverflow.com/questions/25583355/spring-service-default-scope

* 그러나 singleton인 controller에 주입되면 무용지물인데 어떻게 해야 할까?
  + 레퍼런스에서는 Method Injection(위에서 "해결책"에서 안내하는 해결책 중의 하나)으로 해결하라고 함

## Static은?
* 이렇게 모두 singleton이라면 static 을 쓰는 것이 의미가 있을까?
  + 위 Java singleton의 차이라는 부분에서 이 부분을 설명하고 있는데, static의 경우 Classloader안에서 global하기 때문에 차이는 분명히 존재하며, 사용하는데 의의도 있음
  + https://stackoverflow.com/questions/10712897/whats-the-scope-of-a-static-field
  + 개인적인 생각으론 특수한 경우의 web application이 아니라면 불필요할 것 같음
  + Spring Boot는 기본적으로 context가 하나로 되어있기에 쓰일 일이 없다고 보는 것 같음

* 번외로 static 변수에 값을 주입하는 방법은?
  + https://stackoverflow.com/questions/7253694/spring-how-to-inject-a-value-to-static-field
  + 며칠 전 찾아봤던 문제인데, 본문에 링크된 글 중에서도 위 문제를 참조하고 있었음
  + 다들 비슷한 고민을 하는가 보다~

* 그럼 static method(= class method)를 Spring에서 사용하는 것은 어떨까?
  + 몇몇 개발자들 말론 큰 의미는 없다고 함 (상태(state)가 필요없는 곳에서는 사용해도 "무방"한 정도?)
    - https://www.slipp.net/questions/58 
  + 엄청 오래된 글이지만, 상관없다 vs static은 지양
    - https://www.theserverside.com/discussions/thread/23812.html
  + Spring의 IoC개념이 무너지므로 별로라고 생각 (Util이라면 모르겠지만..)
    - https://okky.kr/article/291799
  + 신뢰하기 어려우므로 별로라고 생각
    - http://blog.doortts.com/119
  + 바르게 사용하자
    - http://vaert.tistory.com/101

* 덧붙여, Java static에 대한 자세한 설명은 여기 참고
  + https://www.baeldung.com/java-static
  + (이 사이트는 최근에 발견했는데 굉장히 깔끔하게 설명이 잘 되어 있음, 예전엔 mkyong.com이 많이 검색됐었는데...)

## 결론
요약하면 Spring에서 bean은 기본적으로 singleton이고, stateless하다.  
IoC를 통해 Spring이 lifecycle(=scope)를 관리해준다.  
여기에 맞춰 개발을 하면 고통 받을 일이 없다. 성능도 좋고.  
  
위 글을 작성하며 느낀 점은 "모든 의문은 답은 Spring 레퍼런스에 적혀 있다"는 것이다.  
신기하게도 모르고 보면 안 읽히는 것도 알고 보면 읽힌다는 것!  
영어라서 부담된다면, 직역이 많지만, 번역된 문서를 읽어보자. (문서 버전이 달라 각 문단의 넘버링이 맞지 않음)  
https://blog.outsider.ne.kr/765

