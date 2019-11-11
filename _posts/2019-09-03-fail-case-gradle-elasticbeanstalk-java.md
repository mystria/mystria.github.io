---
layout: post
title:  "금주의 AWS 실패사례 - Gradle로 Elastic Beanstalk에 Java 배포하기"
date:   2019-09-03 20:00:00 +0900
categories: Linux
comments: true
---
# Elastic Beanstalk에 SpringBoot 배포를 하자

## Spring Boot 의 Web Application
Spring Boot를 이용해 web application을 개발할 때 다양한 방법이 존재하겠지만, 대체적으로 stater kit을 사용하게 될 것이고 따라서 embed된 Tomcat을 사용하게 될 것이다. 이는 기존처럼 Spring으로 구현한 web application을 Tomcat서버에 WAR파일 형식으로 배포하는 것과는 다른 방식으로 배포해야 한다는 뜻이다. 즉, Java가 설치된 서버에서 $ java -jar artifact.jar 같은 형식으로 실행해야 한다. 그래서 평소에 쓰던 ElaticBeanstalk Tomcat을 사용할 수 없다.(Spring Boot를 WAR로 build하면 되긴 함)  
Elastic Beanstalk Java 환경에서 JAR(SpringBoot의 artifact)를 한 방에 배포하기 위한 각고의 삽질을 정리하였다.  

## Elastic Beanstalk의 Java environment
Elastic Beanstalk(이하 EB)는 여러 형식의 배포환경(Environment)를 사전 정의하여 지원하고 있다. 대표적인게 Tomcat이고, 이 외에도 Node.js와 Java 등도 지원한다. 우리는 Spring Boot로 개발하여 bootJar로 빌드 했기 때문에 JAR를 실행할 수 있는 Java 환경에 배포하여야 한다. 이 때 Java환경은 Tomcat환경과 달리 별도로 JVM Options을 전달 할 방법을 지원하지 않기 때문에 특별한 방법이 필요하다.  

* JVM Options를 어떻게 전달하지?
  + Java 환경에서 Java application을 실행할 때의 기본은 Procfile을 만드는 것
    - AWS 공식 가이드 - [Environment configurations](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-platform.html)
    - AWS 공식 가이드 - [Procfile 설정](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/java-se-procfile.html)
    - 주의: [Tomcat port를 5000으로 둬야 reverse-proxy인 nginx가 WebApp을 호출할 수 있다](https://stackoverflow.com/questions/54612962/502-bad-gateway-elastic-beanstalk-spring-boot)
  + Procfile에서 Java 환경의 app을 실행하는 방법을 정의해야 함
  + Procfile에 각종 credentials 값을 넣어둘 순 없잖아!
    - Procfile도 SCM(Git 등)에서 관리하고자 하기 때문에 암호나 URL같은 것을 코드에 담아 둘 수 없음
    - EB > Application > Environment > Configuration > Software > Environment properties를 전달해서 사용하고 싶음 
    - 그러나, System의 getenv()으로만 획득 가능
      - name(key)가 무조건 대문자여야함 -> 기존 코드(static 변수 값) 변경 필요
      - 한 땀 한 땀 넣어야 함 -> 피곤
    - 평소에 쓰던 System.getProperty()를 사용할 수 없음
* 뭔가 더 쉬운 방법은 없을까?
  + JAVA_OPTS로 전달하기
    - 보통은 JAVA_OPTS를 쓰면 getenv("JAVA_OPTS") 잘 인식되고, getProperty()로도 잘된다고 함
    - 그런데, **EB에선 안됨**
  + Procfile에서 java를 실행할때 JAVA_OPTS를 전달면 되겠지?
    - \$ java $JAVA_OPTS -jar artifact.jar
    - **안됨**, $JAVA_OPTS를 못찾음 
      > Error: Could not find or load main class $JAVA_OPTS
  + Procfile에 꼼수를 더해서 JAVA_OPTS전달
    - Shell script 파일을 만들어 JAVA_OPTS를 전달하고, 이 파일을 Procfile이 실행
    - https://stackoverflow.com/questions/48026648/beanstalk-java-platform-with-parameters
* 만약 기존 EB environment에 새로운 버전의 artifact를 배포하면?
  + 기존 Java 프로세스가 남아있는 문제
    - Procfile 실행 시 무조건 Java 프로세스를 한 번 죽이고 가자!

## Gradle의 task로 정리하기
  * 예제 코드
    ~~~ java
      bootJar {
        mainClassName = 'com.my.Application'
        baseName = 'myApplication'
        version = '0.1.0'
        artifactName = baseName + '-' + version + '-' + new Date().format('yyyyMMddHHmmssSSS')
        archiveName = artifactName + '.jar'
        enabled = true
        reproducibleFileOrder = true
      }
      
      task generateProcfile(dependsOn: 'bootJar') {
        description = 'Generate Procfile'
        group = 'build'
        println(description)
        doFirst {
          new File("build/libs", "run.sh").text = """
      #!/bin/bash
      killall -9 java
      java \$JAVA_OPTS -jar $bootJar.archiveName
          """
          new File("build/libs", "Procfile").text = """
      web: sh ./run.sh
          """
        }
      }

      task zipProcfile(dependsOn: 'generateProcfile', type: Zip) {
        description = 'Zip the artifact and Procfile'
        group = 'build'
        println(description)
        from 'build/libs/'
        include bootJar.archiveName
        include 'Procfile'
        include 'run.sh'
        archiveName artifactName + '.zip'
        destinationDir(file('/build/libs'))
      }
    ~~~
* 위 예제를 보면
  + bootJar로 artifact를 JAR파일로 만들고
  + run.sh을 만들어서
    - 모든 java 프로세스를 죽이고
    - JAVA_OPTS를 전달하여 Java 실행
  + Procfile을 만들어서
    - run.sh를 실행
  + 위 파일을 zip으로 묶음
* 위 task를 정의하기 위해 아래 글들을 참고
  + Gradle로 파일 생성
    - https://stackoverflow.com/questions/38275583/create-version-txt-file-in-project-dir-via-build-gradle-task
  + Gradle로 파일 압축
    - https://stackoverflow.com/questions/36863456/gradle-task-to-create-a-zip-archive-of-a-directory
  + Shell 명령으로 프로세스 한번에 죽이기 (killall -9 java)
    - https://bakyeono.net/post/2015-05-05-linux-kill-process-by-name.html

## 참고
* Azure에서는 JAVA_OPTS전달 후 System.getProperty()가 동작하는 듯?
  + https://blogs.msdn.microsoft.com/azureossds/2015/10/09/setting-environment-variable-and-accessing-it-in-java-program-on-azure-webapp/

* System properties와 Environment variables 의 차이
  + https://stackoverflow.com/questions/7054972/java-system-properties-and-environment-variables
  + https://www.baeldung.com/java-system-get-property-vs-system-getenv

* ElasticBeanstalk Procfile 배포
  + https://medium.com/@autumn.bom/deploying-spring-boot-jar-application-on-beanstalk-java-se-platform-45d8d04608ae

