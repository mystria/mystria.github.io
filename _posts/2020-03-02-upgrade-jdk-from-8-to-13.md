---
layout: post
title:  "JDK를 버전 8에서 버전 13으로 올리기"
date:   2020-03-02 18:00:00 +0900
categories: Java, SpringBoot
comments: true
---
# JDK 버전 업그레이드
살다보면 오래된 JDK로 더 이상 버틸 수 없을 때가 있다.  
JDK 버전을 업그레이드 하면서 Spring Boot기반 개인 프로젝트에서 변경되는 것들 정리해 봤다.  
실제로는 변경 요소가 본문 보다 훨씬 더 많을 것이다.  

## OpenJDK 13 설치
* JavaSE(Oracle JDK)는 약 버전 9 부터 유료 라이선스로 변경됨
  + [관련하여 참고하기 좋은 글](https://goddaehee.tistory.com/183)
* (Windows의 경우)Java 폴더에 JDK파일 다운로드(복사) 후 Environment Variables에서 JAVA_HOME 변경
* (Ubuntu의 경우)apt-get 으로 설치 후 기본 JDK 선택
  + PPA: https://launchpad.net/~openjdk-r/+archive/ubuntu/ppa
  + Install: https://installvirtual.com/how-to-install-openjdk-13-on-ubuntu-19/
* 버전 확인
~~~ sh
$ java --version
openjdk 13.0.2 2020-01-14
OpenJDK Runtime Environment (build 13.0.2+8)
OpenJDK 64-Bit Server VM (build 13.0.2+8, mixed mode, sharing)
~~~

## 코드 외 부분
* 구버전 Gradle이 지원을 하지 않음
  + 정확한 버전 지원은 모르겠으나 Gradle 버전 5 이상(또는 6 이상)만 지원
    - 버전 4 이하는 에러 표시2
    ``` sh
    $ gradle -version
    FAILURE: Build failed with an exception.
    
    What went wrong:
    Could not determine java version from '13.0.2'.
    ```
  + 주의: Gradle의 plugin 중에서도 아직 지원하지 않는 경우가 존재
* IDE 설정 필요
  + IDE에 연결되어 있는 Java home과 Gradle home 등을 바뀐 환경에 맞게 수정 필요
  + org.gson 에서 java.sql.Time class 못찾는 문제 발생
    - 왠지 모르겠지만, IDE에서 Shorten command line 값을 JAR manifest로 변경 필요
    - [참고](https://stackoverflow.com/a/51591417/8350542)

## 코드 내 부분
* Gradle 설정 변경 필요
  + Java plugin을 사용 중이라면 sourceCompatibility 등을 Java 13으로 변경 필요
    - [상수인 JavaVersion.VERSION_13 를 신규 Gradle 버전이 지원](https://docs.gradle.org/current/javadoc/org/gradle/api/JavaVersion.html)
  + Gradle 버전이 올라가면서 관련된 plugin이 변경됨
    - [버전 5 부터 FindBugs plugin이 deprecated됨](https://docs.gradle.org/current/userguide/upgrading_version_5.html#the_findbugs_plugin_has_been_removed)
    - [SpotBugs](https://spotbugs.github.io/)를 대안(spiritual successor)으로 제안
* Spring Boot 버전 업그레이드 필요
  + Sprinb Boot 2.2.x 이상만 지원
  + Spring Boot 버전이 올라가면서 관련된 라이브러리 업그레이드 필요
    - redis.clients 를 3.1 이상, [참고](https://stackoverflow.com/q/33128318/8350542)
      - RedisOperationsSessionRepository가 deprecated됨 -> RedisIndexedSessionRepository 사용
    - Redis 관련 코드 수정 필요
      - @EnableRedisHttpSession 사용 시, session 값 deserialize 오류 발생
      ~~~ sh
      Cannot deserialize; nested exception is org.springframework.core.serializer.support.SerializationFailedException: Failed to deserialize payload
      ~~~
      - 신규 SpringBoot 버전에서는 해당 annotation이 불필요한듯, [삭제 필요](https://stackoverflow.com/a/38696205/8350542)
      - 참고로 EnableRedisHttpSession 사용 시에는 session timeout 설정 위치를 EnableRedisHttpSession annotation의 maxInactiveIntervalInSeconds 속성으로 정의 해야함, 미사용시엔 application.properties의 server.servlet.session.timeout 속성
* javax package가 jakarta 로 변경 됨, [OpenJDK 11부터 JavaEE 삭제](http://openjdk.java.net/jeps/320)
  + jakarta.xml.bind-api 또는 javax.xml.bind 의존성 추가 필요
  + [참고](https://stackoverflow.com/a/43574427/8350542)
* SessionId 생성관련 Warning 발생
  + org.apache.catalina.util.SessionIdGeneratorBase.createSecureRandom Creation of SecureRandom instance for session ID generation using \[SHA1PRNG\] took \[000\] milliseconds.
  + [난수 생성시 문제](https://lng1982.tistory.com/261)
* Jackson 관련 Warning 발생
  + JacksonAutoConfiguration$JodaDateTimeJacksonConfiguration - Auto-configuration of Jackson's Joda-Time integration is deprecated in favor of using java.time (JSR-310)

## 느낀점
* 높은 버전이 무조건 좋지는 않음
  + JDK 하위 호환성에서 문제가 발생
  + 라이브러리, 플러그인 등에서 아직 Java 9 초과는 지원 안되는 경우가 존재
* 개선이 많이 됨
  + 개인적으로는 필요했던 Lambda 함수가 추가되어 좋음
  + StackOverflow의 최신 답변에 맞게 적용가능한 Java및 SpringBoot!
* 어쨋든 변화에 적응 필요

## 참고
* [How to set up a spring-boot application with java 13](https://stackoverflow.com/a/58624755/8350542)
* [Maven Repository](https://mvnrepository.com/artifact/org.springframework.session/spring-session-data-redis/2.2.1.RELEASE)

