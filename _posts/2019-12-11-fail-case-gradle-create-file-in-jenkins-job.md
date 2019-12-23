---
layout: post
title:  "금주의 실패사례 - Jenkins job이 Gradle build시 file 생성"
date:   2019-12-11 16:00:00 +0900
categories: Jenkins Gradle
comments: true
---
# Gradle에서 file 생성이 Jenkins job에서 안됨
일부 과제에서는 완성된 jar파일과 몇 몇 추가 파일을 묶어서 zip으로 만드는 작업이 필요했다. 그래서 gradle.build에서 파일을 생성하게 하였더니 잘 동작하였다.  
Jenkins에서는 build할 때 gradlew로 동작하게 설정하였고, build job을 실행했더니?  
권한 문제로 파일이 만들어지지 않는 에러가 발생했다.  

## 증상
* 에러 발생
  + No such file or directory 에러가 발생
    ~~~ ssh
    attempting to create file: /home/jenkins/server-name/workspace/somepath\somefile.txt
    FATAL: No such file or directory
    java.io.IOException: No such file or directory
        at java.io.UnixFileSystem.createFileExclusively(Native Method)
        at java.io.File.createNewFile(File.java:947)
        at java_io_File$createNewFile.call(Unknown Source)
        ...
    ~~~
* 에러 발생 코드
  + build.gradle
    ~~~ groovy
    task generateProcfile(dependsOn: 'bootJar') {
      description = 'Build Jar'
      group = 'build'
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
    ~~~

## 해결
* gradle create file 로 검색
  + 별다른 이유나 해결책은 못찾고, 다른 library를 써야 된다 같은 답글만 찾음
    - https://javadoc.jenkins-ci.org/hudson/FilePath.html
  + 그러다 다른 사람의 예제를 보니 다른 방식으로 파일 생성 중
    - https://stackoverflow.com/a/52976567/8350542
    - 따라 해보자..
  + build.gradle
    ~~~ groovy
    task generateProcfile(dependsOn: 'bootJar') {
      description = 'Build Jar'
      group = 'build'
      doFirst {
        file("$buildDir/libs").mkdirs()
        file("$buildDir/libs/run.sh").write("""
    #!/bin/bash
    killall -9 java
    java \$JAVA_OPTS -jar $bootJar.archiveName
        """)
        file("$buildDir/libs/Procfile").write("""
    web: sh ./run.sh
        """)
      }
    }
    ~~~
* file()은 왜 된걸까?
  + .text = 말고, .write 를 쓴 탓? 진실은 저 너머에...

## 참조
* https://stackoverflow.com/questions/30608444/in-jenkins-job-create-file-using-system-groovy-in-current-workspace
* https://stackoverflow.com/questions/53634042/jenkinsfile-create-a-new-file-groovy
* https://stackoverflow.com/questions/45338959/how-to-create-and-write-to-file-from-jenkins-with-groovy-scripting

* https://stackoverflow.com/questions/52972830/programmatically-create-a-file-in-gradle-build-script
