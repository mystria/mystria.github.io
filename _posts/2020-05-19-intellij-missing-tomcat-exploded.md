---
layout: post
title:  "금주의 실패사례 - IntelliJ 버전 업 후 Tomcat에서 exploded artifact 실종"
date:   2020-05-19 17:00:00 +0900
categories: IntelliJ Java Gradle
comments: true
---

# IntelliJ에서 exploded된 artifact를 못찾는 문제
무슨 바람이 불었을까? 한참 잘 쓰던 IntelliJ를 2020.1.1로 업그레이드 했다.  
이것 저것 좀 더 좋아지려나 했더니, 왠걸.. 잘되던 Tomcat 실행만 안된다.  
잘 되던 기능인데 업데이트 후 갑자기 안된다면 업데이트가 문제일 것이다. 그러나 IntelliJ issue를 찾다보니 몇 년 전 부터 발생하던 고질적인 문제였다.  

## 문제 분석
* Tomcat project 실행이 안되는 이유를 확인해보니 exploded artifact가 만들어지지 않아 못 찾는 것
  + Project Structure 상에는 정의되어 있지만, 빌드 시 생성되지 않음
* [gradle war exploded is not generated when 'Delegate IDE build/run actions to gradle' is enable](https://youtrack.jetbrains.com/issue/IDEA-176700)
* 에러 로그
  ~~~
  Connected to server
  [2020-05-17 04:04:05,399] Artifact Gradle : com.test : your.web-1.0.2.20200516011223.war (exploded): Artifact is being deployed, please wait...
  [2020-05-17 04:04:05,431] Artifact Gradle : com.test : your.web-1.0.2.20200516011223.war (exploded): Error during artifact deployment. See server log for details.
  [2020-05-17 04:04:05,439] Artifact Gradle : com.test : your.web-1.0.2.20200516011223.war (exploded): com.intellij.javaee.oss.admin.jmx.JmxAdminException: com.intellij.execution.ExecutionException: C:\project\your.web\build\libs\exploded\your.web-1.0.2.20200516011223.war not found for the web module.
  ~~~

## IntelliJ의 Tomcat project 실행 방식
* Spring MVC로 만들어진 artifact(= WAR파일)를 IDE상의 Tomcat으로 실행하고자 함
  + IDE의 Run/Debug를 이용
  + 실제 build된 WAR를 불러와서 실행할 수도 있지만, 그럴 경우 디버깅이 안됨(= 그냥 Tomcat에 실행한 것)
  + IntelliJ의 Project Structure > Artifact 에서 WAR를 exploded하게 정의 가능
    - exploded: Servlet container에 로드된 것 처럼 (압축을 풀어둔 것 처럼) 바이너리를 IDE에 배포해 둠
  + Run/Debug Configuration 에서 실행(Run)할 deployment를 지정할 때 이 artifact를 지정
    - /build/libs/exploded 에 생성됨
    - exploded 상태의 artifact를 선택 필요
    - 이를 통해 IDE가 직접 Tomcat 위에 WebApp을 실행하고 디버깅 가능

## 해결책
* Workaround로 Gradle을 이용하라는 제안
  + [2017.2.3: Gradle exploded war is not unpacked to Tomcat (Artifact: Web exploded)](https://youtrack.jetbrains.com/issue/IDEA-178450#focus=streamItem-27-4068591.0-0)
    - 이 방법은 Gradle task로 exploded를 직접 생성함
    - 이 대로 해보면 어쨋든 실행은 잘 되지만, 그냥 WAR를 실행한 것 처럼 디버깅이 안됨
* IntelliJ의 Gradle설정을 변경하라는 제안
  + [Intellij IDEA 19.3.3 hot deployment and update of web resources fails](https://youtrack.jetbrains.com/issue/IDEA-234209#focus=Comments-27-3969024.0-0)
    - Settings > Build, Execution, Deployment > Build Tools > Gradle > Build and run using
    - Default로 되어있던 Gradle에서 IntelliJ IDEA로 변경

## IntelliJ의 Gradle 설정 변경으로 해결
* IDE생각: Project가 Gradle로 정의되어있음 → Gradle로 실행
* Gradle로 정의된 프로젝트여서 Gradle 설정으로 실행을 하니 exploded가 생성되지 않은 것
  + 그래서 수동으로 만들라고 한건가?
* Build and run 설정을 IntelliJ IDEA로 지정해두면 Project Structure대로 실행 → exploded를 생성
  + 버전 업을 하면서 이 설정의 기본 값이 Gradle로 변경된건가?
* 결과
  ~~~
  [2020-05-17 04:06:54,915] Artifact Gradle : com.test : your.web-1.0.2.20200516011223.war (exploded): Artifact is deployed successfully
  [2020-05-17 04:06:54,916] Artifact Gradle : com.test : your.web-1.0.2.20200516011223.war (exploded): Deploy took 26,972 milliseconds
  ~~~
  

