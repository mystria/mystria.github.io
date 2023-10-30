---
layout: post
title:  "Maven 에서 받은 Runnable JAR 를 실행하는 Gradle Task"
date:   2023-04-04 13:00:00 +0900
categories: Java Gradle Maven JAR
comments: true
---

# Gradle Task 에서 Maven 에서 받은 Runnable JAR 를 실행하자

Gradle 에서 이런저런 task 를 수행해야 할 때, 가끔 python script 나 java application 을 실행해야 할 때가 있다.

만약 Java application(runnable JAR) 을 task 로 실행시키려면, 그리고 이 task 를 특정 task 이전에 수행시키려면 어떻게 해야하는지 알아보자.

참고로 본 Gradle script 는 Gradle 8.0.2 기준으로 작성되었다.

# Task 정의

일단 Java 를 실행시킬 task 는 다음과 같이 정의한다.

```groovy
// 첫번째 방식
tasks.register("newTask", JavaExec) {
}

// 두번째 방식
task newTask(type: JavaExec) {
}
```

위 두가지 방법 중 두 번째 방법이 간편해서 많이 사용되지만, 정적 코드 분석에서는 안전한 정의를 위해 첫 번째 방법을 권장한다.

```groovy
dependencies {
    compileOnly 'com.project:com.project.yours:1.0.0'
}

tasks.register("newTask", JavaExec) {
    setMainClass 'com.project.yours.Main'
    classpath sourceSets.main.compileClasspath
}
```

compileOnly 로 runnable JAR 를 가져오면 (확실하진 않지만) 이는 library 가 아니므로 의존성이 제대로 로드되지 않는다. 즉, 별도로 정의하지 않으면 runnable JAR 가 필요한 의존성들을 가져오지 않는다. 이 상황에서 classpath 를 compileClasspath 로 지정한다면 Caused by: java.lang.ClassNotFoundException 에러를 마주하게 된다. 또한 runtimeClasspath 로 해도 마찬가지다.

그래서 runtimeOnly 혹은 implementation 으로 runnable JAR 를 가져오면 정상적으로 - runnable JAR 가 필요로하는 의존성들 포함하여 - 의존성이 로드된다. 이로써 runtimeClasspath 를 사용할 수 밖에 없어진다.

runtimeClasspath 상에서 새로 정의한 task 를 수행시키면 잘 동작한다.

```groovy
dependencies {
    runtimeOnly 'com.project:com.project.yours:1.0.0'
}

tasks.register("newTask", JavaExec) {
    setMainClass 'com.project.yours.Main'
    classpath sourceSets.main.runtimeClasspath
}
```

# DependsOn 설정

내가 정의한 새 task 가 다른 task 이전에 수행되어야 한다면 다음과 같이 dependsOn 을 활용할 수 있다.

```groovy
otherTask.dependsOn newTask

otherTask.dependsOn tasks.named("newTask")
```

첫 번째가 깔끔하긴한데, newTask 를 인식못할 경우 찜찜할 수 있으므로 두 번째 방법을 사용한다.

만약 otherTask 가 compileJava 이 후에 수행되어야 하는데, 이 보다 먼저 수행될 newTask 가 runtimeClasspath 를 참조할 경우 Circular dependency 이슈가 발생할 수 있으니 주의하자.

runtimeClasspath 가 compileJava 선행을 요구하여 otherTask → compileJava → newTask → compileJava 로 task chain 이 생성되기 때문이다.

[https://stackoverflow.com/questions/19529817/how-to-execute-a-task-of-type-javaexec-before-compilejava](https://stackoverflow.com/questions/19529817/how-to-execute-a-task-of-type-javaexec-before-compilejava)

다행이도 이 때는 newTask 를 compileClasspath 상에서 수행하면 해결된다.

# 의존성 문제

잠깐, 첫 번째에서 runtimeClasspath 를 사용해야 한다고 했는데, Circular dependency 때문에 compileClasspath 를 사용해야 한다면?

runnable JAR 를 compileClasspath 에서 동작시키기 위해서는 꼼수가 필요하다. 즉, compile 단계에서 runnable JAR 가 필요로 하는 의존성들을 준비하는 것이다.

- [https://stackoverflow.com/a/7112419/8350542](https://stackoverflow.com/a/7112419/8350542)
- [https://discuss.gradle.org/t/running-an-executable-jar-from-a-remote-maven-repo/15444/2](https://discuss.gradle.org/t/running-an-executable-jar-from-a-remote-maven-repo/15444/2)

꽤 오래 전(2011년)에 제시된 해결책이지만, 잘 동작한다.

```groovy
configurations {
    yourProject // 프로젝트 이름 등으로 정의
}
dependencies {
    yourProject 'com.project:com.project.yours:1.0.0'
}
tasks.register("newTask", JavaExec) {
    setMainClass 'com.project.yours.Main'
    classpath configurations.yourProject, sourceSets.main.compileClasspath
}
```

참고로 task 의 classpath 는 여러개 지정 가능하다.

configurations.yourProject 로 정의한 runnable JAR 의 의존성 위에서 task 를 실행시킨다. 그리고 추가로 sourceSets.main.compileClasspath 를 classpath 에 지정해주면, 현재 내 project 의 의존성을 함께 이용하여 application 이 실행된다. 

오직 JAR 에 포함된 라이브러리만 활용할 때는 transitive 하지 않은 의존성들은 누락될 수 있기 때문에 경우에 따라서 의존성 부족 문제가 발생할 수 있다. 추가적인 classpath 를 지정해 주자.

다시 말하지만, circular dependency 문제가 없다면 그냥 sourceSets.main.runtimeClasspath 를 사용할 수 있다. 실행하려는 runnable JAR 가 어떤 시점에 어떤 의존성들과 함께 수행되어야 하는지 잘 판단해서 지정하자.

# 정리

뭔가 복잡하고 불필요해 보이지만, 가끔 이런 설정이 필요할 때가 있으니 알아두자.