---
layout: post
title:  "금주의 AWS 실패사례 - Gradle로 ElasticBeanstalk Java 파일 생성하기"
date:   2019-07-26 20:00:00 +0900
categories: Linux
comments: true
---
# ElasticBeanstalk에 SpringBoot 배포를 하자

SpringBoot를 구현 - Tomcat이 내장됨
평소에 쓰던 ElaticBeanstalk Tomcat을 못 사용함
Java를 사용하고자 함
bootJar를 통해 빌드하면 jar 파일이 자동으로 실행됨

그런데, JVM Options를 어떻게 전달하지?

제일 기본은 Procfile을 만들고 java로 jar를 실행할 때 전달하는 것,

Procfile에 각종 credential을 넣어둘순 없잖아!

Java의 Environment properties를 전달해서 사용하고 싶음 - 그러나, 이건 사실 System의 getenv()

평소에 쓰던 System.getProperty()를 사용할 수 없음...
고육지책으로 getenv를 사용하고자 했으나, name(key)가 무조건 대문자여야함... 기존 static 변수 값 변경 필요

뭔가 더 쉬운 방법은 없을까?

그래 JAVA_OPTS로 전달하면 되지!!

보통은 JAVA_OPTS를 쓰면 getenv("JAVA_OPTS") 잘 인식되고, getProperty()도 잘된다고 함
그런데, EB에선 안됨

그러면 Procfile에서 java를 실행할때 JAVA_OPTS를 전달하면 되지....
안됨 ㅋ

꼼수로 해결

앗 근데, 재 배포하면, 기존 java 프로세스가 남아있네!

무조건 한번 죽이고 가자!


이렇게하면 되기는 한데.. 이걸 gradle task로 어떻게 만들지?

각고의 노력 끝에 아래와 같이 완성!

~~~

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
  }
  new File("build/libs", "Procfile").text = """
web: sh ./run.sh
    """
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


##
AWS 공식 가이드 - Environment configurations
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-platform.html
AWS 공식 가이드 - procfile 설정
https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/java-se-procfile.html
(port를 5000으로 둬야 reverse-proxy인 nginx가 WebApp을 호출할 수 있다)
https://stackoverflow.com/questions/54612962/502-bad-gateway-elastic-beanstalk-spring-boot


JAVA_OPTS를 java에 넘기기 위해서는 다음과 같이 꼼수를 써야한다.
https://stackoverflow.com/questions/48026648/beanstalk-java-platform-with-parameters

Gradle 파일 생성
https://stackoverflow.com/questions/38275583/create-version-txt-file-in-project-dir-via-build-gradle-task
Gradle 파일 압축
https://stackoverflow.com/questions/36863456/gradle-task-to-create-a-zip-archive-of-a-directory

Azure에서는 JAVA_OPTS전달 후 System.getProperty()가 동작하는 듯?
https://blogs.msdn.microsoft.com/azureossds/2015/10/09/setting-environment-variable-and-accessing-it-in-java-program-on-azure-webapp/


System properties와 environment variables 의 차이
https://stackoverflow.com/questions/7054972/java-system-properties-and-environment-variables
https://www.baeldung.com/java-system-get-property-vs-system-getenv


그냥 참고
ElasticBeanstalk Procfile 배포
https://medium.com/@autumn.bom/deploying-spring-boot-jar-application-on-beanstalk-java-se-platform-45d8d04608ae

프로세스 한번에 죽이기
https://bakyeono.net/post/2015-05-05-linux-kill-process-by-name.html
(killall -9 java)
