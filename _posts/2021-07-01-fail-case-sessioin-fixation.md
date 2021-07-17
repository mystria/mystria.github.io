---
layout: post
title:  "금주의 실패사례 - Session Fixation 해결하기"
date:   2021-07-01 20:00:00 +0900
categories: Java Spring Session Security
comments: true
---

# Session Fixation 해결하기
Session Fixation, 세션 고정 취약점은 무엇일까?  
간단히 요약하면 FrontEnd에서 BackEnd로 request할 때 쓰는 sessionId를 뻐꾸기 알처럼 이용한 누군가에 의해서 권한을 빼앗길 수 있는 문제이다.  
보통 처음 사이트에 접속하면 sessionId key(이하 key)를 발급 받는데, 이 key를 이용하여 로그인 하고, 서비스를 이용하게된다. 만약 이 key가 내가 서비스로부터 직접 발급받은 것이 아닌 누군가 미리 발급받아 몰래 넣어둔 것이라면? 그 key로 내가 로그인 해버리 순간 그 권한을 침투자도 사용할 수 있게되는 것이다. 핵심은 '첫 접속 시 발급받은 key' 이다. 이미 로그인 된 key를 훔쳐가는 부분은 CSRF나 CORS의 영역.  
이런 트릭이 있다는 것을 알게된 건 얼마 전의 일이고, 역시 Spring에서는 이를 간단히 해결해 준다. 근데 나는 안된다...?

## Session Fixation 해결책
  + 로그인 후 sessionId를 새로 발급해주면 해결(간단!)
  + 확인해보니 Spring 기본 설정으로 방어 로직(session migration)이 적용되어 있다는데?

## 문제
* 증상확인
  + 최초 접속 시 미인증(unautenticated) 유저에게 sessionId key가 발급됨
  + 로그인 후 이 key가 유지됨 ← SessionFixation
    - 로그인 성공시 key가 변경되는게 정상 동작
* 본인의 환경
  + Spring Boot 2.2.0
  + 특정 container부터 해당 기능이 적용되는 건가?

## 해결 과정
* 작업 순서
  1. 혹시나 하여 .sessionFixation().migrateSession() 을 명시적으로 적용 → 안됨
  2. .migrateSession() 대신 좀 더 높은 버전에서 지원하는 .changeSessionId() 적용 → 안됨
  3. SessionFixationProtectionStrategy 라는게 있다?
      - .sessionAuthenticationStrategy(new SessionFixationProtectionStrategy()) 적용 → 안됨
      - 이것도 default로 적용되어 있음
  4. 그렇다면 다른 strategy 적용,
      - .sessionAuthenticationStrategy(new ChangeSessionIdAuthenticationStrategy()) 적용 → 안됨
  5. 혹시 AuthenticationProcessingFilter(실제 로긴 처리하는 로직)의 순서가 잘못되었나?
      - 앞에? .addFilterBefore(processingFilter, SessionManagementFilter.class) 적용 → 안됨
      - 그럼 뒤? .addFilterAfter(processingFilter, SessionManagementFilter.class) 적용 → 안됨

* 뭐가 문제일까?
  + 처음부터 확인해보기 위해 .sessionFixation().none() (=미적용)을 실행 → 기존 동작과 동일
  + 그렇다면, 처음부터 .sessionFixation()설정이 안먹히는건가?
  + 완전히 수동으로 처리하는 방법도 있는 것 같아서 시도
    ~~~ java
      // Clear sessionId
      HttpSession session = request.getSession(false);
      if (session != null && !session.isNew()) {
        session.invalidate();
      }
      session = request.getSession(true); // create the session
    ~~~
  + 최초 접속하여 session이 발급되는 지점에서 기존 sessionId가 있으면 삭제하고(invalidate) 재발급하는 로직
  + 이거는 그냥 강제 로그아웃 시키는(기존 sessionId를 삭제)것과 동일하므로 무의미하다고 판단
  + 로그인 후 이 로직을 실행 시키면 된다고 하는데, 찜찜해서 마지막에 시도해보는 걸로...

* 여러 해결책을 찾던 중 다음에서 힌트 획득
  + https://stackoverflow.com/a/58758021/8350542
    ~~~ xml
      <bean id="authenticationFilter" class="org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter">
          ...
          <property name="sessionAuthenticationStrategy" ref="sessionFixationProtectionStrategy" />
          ...
      </bean>
    ~~~
  + sessionFixationProtectionStrategy를 **Filter에 적용**해야 한다!

* 코드 확인
  - 본인은 AuthenticationProcessingFilter를 상속 후 custom하여 사용 중
  - 본인의 AuthenticationProcessingFilter를 찾아가 안에 AbstractAuthenticationProcessingFilter를 확인해 보니 실제로 setSessionAuthenticationStrategy() 항목이 존재
  - 여기엔 기본적으로 NullAuthenticatedSessionStrategy (=아무것도 안함)이 적용되어 있음
* 코드 수정
  - AuthenticationProcessingFilter의 constructor에서 아래를 적용 → **성공**
    > setSessionAuthenticationStrategy(new SessionFixationProtectionStrategy());

## 확인
실제로 첫번째 접속시 발생하는
> Set-Cookie: SESSIONID=NTIzNDkzZmEtNDAyMi00YjM0LWI0MzItMmJhNjViZDY1MWVm

의 값을 받지만, 로그인 완료하는 지점에서
> Set-Cookie: SESSIONID=NzBhMWIyMjMtOTU2NS00NGIwLThhM2EtZjRjYTg5YTc5MjVh

처럼 한 번 더 받는 것을 확인할 수 있다. 즉, sessionId key가 변경된다는 것이다.

## 정리
* Custom AuthenticationProcessingFilter 사용 시 Security Config의 sessionManagement의 설정이 동작하지 않을 수 있음
* 다르게 말하면, 직접 만든 AuthenticationProcessingFilter를 사용한다면, 별도로 strategy를 적용해주어야 함
* Filter에 해당 로직을 넣지 말고,
  > oAuth2ProcessingFilter.setSessionAuthenticationStrategy(new SessionFixationProtectionStrategy());

  > http.addFilterBefore(oAuth2ProcessingFilter, UsernamePasswordAuthenticationFilter.class);

  이런 방식(WebSecurityConfigurerAdapter에서 HttpSecurity의 config)도 가능

## 참고
* 왜 session id가 login 전에 생성되는가? - 로그인 후 돌아오기 위해서, 이 부분 때문에 session fixation 방지 기술이 필요해짐
  + https://stackoverflow.com/questions/22507590/why-is-a-new-session-created-before-i-login
* Session-Fixation attack이란 무엇인가? - 해당 페이지의 이미지를 읽어보면 바로 이해가 됨, Spring 기본 설정도 알려 줌
  + https://www.javadevjournal.com/spring-security/spring-security-session-fixation/
  + https://guleum-zone.tistory.com/163
  + https://www.baeldung.com/spring-security-session#session-fixation
* OWSAP의 SessionFixation 정의
  + https://owasp.org/www-community/attacks/Session_fixation
* 스프링 없이 Session-Fixation막는 방법(수동 코딩)
  + https://stackoverflow.com/a/8165963/8350542
* 스프링 세션 관리에 대한 한글 설명
  + https://velog.io/@devsh/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%84%B8%EC%85%98-Session-Management-%EC%BB%A4%EC%8A%A4%ED%84%B0%EB%A7%88%EC%9D%B4%EC%A7%95-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0
