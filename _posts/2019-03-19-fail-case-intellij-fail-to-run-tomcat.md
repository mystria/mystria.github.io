---
layout: post
title:  "금주의 실패사례 - IntelliJ에서 Tomcat이 실행되지 않음"
date:   2019-03-19 16:00:00 +0900
categories: IntelliJ Tomcat
comments: true
---

# IntelliJ가 왜 이러지?
개발환경(IDE)으로 예전에는 Eclipse를 썼지만, 회사에서 IntelliJ를 지원해 준 이후로 IntelliJ를 쓰고 있다.  
단점으로 아무래도 비싼 가격이겠지만, 그에 상응하는 장점으로 성숙한 UX, 좋은 plugin 들, 그리고 고객지원(?) 이다.  
  
강력한 도구로서 잘 쓰고 있었지만, 갑자기 Tomcat을 띄우는데, 자꾸 실패를 하는 경우가 발생했다.  
잘되다가.. 왜 그럴까?  
수 많은 검색 끝에 간단한 해결책을 찾아 작성해 둔다.

## 에러 메시지
~~~ sh
"C:\Apache Software Foundation\Tomcat8\apache-tomcat-8.5.35\bin\catalina.bat" run
Using CATALINA_BASE:   "C:\Users\XXX\.IntelliJIdea2018.3\system\tomcat\Tomcat_8_5_test-project"
[2019-03-19 02:54:17,014] Artifact Gradle : project : test-project.war (exploded): Waiting for server connection to start artifact deployment...
Error opening zip file or JAR manifest missing : C:\Users\XXX\.IntelliJIdea2018.3\system\captureAgent\debugger-agent.jar
Using CATALINA_HOME:   "C:\Apache Software Foundation\Tomcat8\apache-tomcat-8.5.35"
Using CATALINA_TMPDIR: "C:\Apache Software Foundation\Tomcat8\apache-tomcat-8.5.35\temp"
Using JRE_HOME:        "C:\Program Files\Java\jdk1.8.0_151"
Using CLASSPATH:       "C:\Apache Software Foundation\Tomcat8\apache-tomcat-8.5.35\bin\bootstrap.jar;C:\Apache Software Foundation\Tomcat8\apache-tomcat-8.5.35\bin\tomcat-juli.jar"
Error occurred during initialization of VM
agent library failed to init: instrument
~~~
  * 메시지 분석
    + catalina 실행 시, debugger-agent.jar 를 열 수 없다고 함
    
## 여러가지 시도
  * CATALINA_BASE 를 지워보기
    + Catalina의 베이스 위치는 IntelliJ에서 프로젝트별로 tomcat파일을 생성해 둔 곳
      - 이 곳에서 뭔가가 충돌나서 debugger-agent를 읽지 못하지 않을까 하여 통채로 삭제
      - Tomcat 환경을 실행 시킬때 마다 재생성 됨
    + 가끔 동작함
  * debugger-agent.jar 지워보기
    + 지워도 Tomcat을 실행 시킬때 마다 재생성 됨
    + 효과 없음
  * Run/Debug Configurations의 Tomcat Server 재생성
    + 첫번째 시도와 비슷한 맥락으로 도전
    + 가끔 동작함

## 해결책
  * 위에서 말했듯, 가끔 동작함
    + "가끔 동작"한다는 말은 다시말해 저 시도들과 무관 한 것이 아닐까?
  * IntelliJ의 장점으로 언급했듯, 여러가지 키워드로 검색하다보니 아래와 같은 고객지원 글 확인
    + 해결책은 있지만, 원인은 못찾은 듯
      - https://intellij-support.jetbrains.com/hc/en-us/community/posts/360000166640-Running-Debug-in-Java-can-t-find-CaptureAgent
    + Settings > Build, Execution, Deployment > Debugger > Async Stacktraces 를 비활성화 시키면 됨
