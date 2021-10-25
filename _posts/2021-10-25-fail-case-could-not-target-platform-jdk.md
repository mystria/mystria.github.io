---
layout: post
title:  "'Java SE *' using tool chain: 'JDK * (1.*)' 에러"
date:   2021-10-25 20:00:00 +0900
categories: Java Jenkins CICD
comments: true
---

# Jenkins에서 Java 빌드 시 발생한 JDK 버전 문제

## 발단

Jenkins에서 Java project 빌드 시 아래와 같은 에러가 발생.

```bash
Starting a Gradle Daemon (subsequent builds will be faster)
> Task :compileJava FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileJava'.
> Could not target platform: 'Java SE 13' using tool chain: 'JDK 8 (1.8)'.

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.2/userguide/command_line_interface.html#sec:command_line_warnings

BUILD FAILED in 1m 37s
2 actionable tasks: 2 executed
```

현재 '사용 중'인 JDK 8로 Java 13을 빌드하려니까 안된다는 뜻 인데, 이 project는 gradle로 Java 버전을 정의해두고 있다. 실제로 빌드하려는 JDK와 project의 Java 버전이 안맞으면(지원이 불가능하면) 수시로 볼 수 있는 에러이다.

```groovy
java {
	sourceCompatibility = JavaVersion.VERSION_13
	targetCompatibility = JavaVersion.VERSION_13
}
```

```bash
declare -x JAVA_HOME="/usr/lib/jvm/java-8-oracle"
```

특이할 점은 해당 Jenkins를 여러개의 node로 운영중인데, primary(또는 master)나 secondary 중 한 곳에서 발생했다는 것이다. 특정 node에서만 실패할 뿐 보통은 빌드가 성공한다.  이 '보통의 경우'에는 JAVA_HOME이 달라도 빌드가 성공하는데, 왜 어떤 곳은 실패하는 것일까? 

## 분석

Google에 검색해보면 IntelliJ에서 주로 발생하는 문제인 것 같고, 해결책도 IntelliJ해결책만 나옴

해결 요점은 IntelliJ(또는 IDE)의 project의 SDK 버전을 맞춰줘야 한다는 것.

- IntelliJ의 Project Settings에 JDK버전 설정: [https://twofootdog.tistory.com/57](https://twofootdog.tistory.com/57)
- 마찬가지로 Gradle JVM 설정: [https://stackoverflow.com/questions/43995886/gradle-could-not-target-platform-java-se-8-using-tool-chain-jdk-7-1-7](https://stackoverflow.com/questions/43995886/gradle-could-not-target-platform-java-se-8-using-tool-chain-jdk-7-1-7)

이 외에도 많은 해결책이 제시되고 있지만 해결되지는 않았다.

- Java 버전을 맞춰라: [https://stackoverflow.com/questions/57514259/could-not-target-platform-java-se-11-using-tool-chain-jdk-8-1-8](https://stackoverflow.com/questions/57514259/could-not-target-platform-java-se-11-using-tool-chain-jdk-8-1-8)
- Java 버전을 맞춰라: [https://unix.stackexchange.com/questions/149452/java-location-from-usr-bin-java](https://unix.stackexchange.com/questions/149452/java-location-from-usr-bin-java)
- gradle 버전을 맞춰라: [https://stackoverflow.com/questions/64153532/gradle-could-not-target-platform-java-se-11-using-tool-chain-jdk-8-1-8](https://stackoverflow.com/questions/64153532/gradle-could-not-target-platform-java-se-11-using-tool-chain-jdk-8-1-8)
- groovy 버전을 맞춰라: [https://stackoverflow.com/questions/46126054/gradle-compilegroovy-java-lang-exceptionininitializererror](https://stackoverflow.com/questions/46126054/gradle-compilegroovy-java-lang-exceptionininitializererror)
- JAVA_HOME을 맞춰라: [https://stackoverflow.com/questions/47500674/gradle-java9-could-not-target-platform-java-se-9-using-tool-chain-jdk-8-1](https://stackoverflow.com/questions/47500674/gradle-java9-could-not-target-platform-java-se-9-using-tool-chain-jdk-8-1)

그러나 사실 위 해결책들이 맞다. 해결을 하고난 뒤에 확인해보니 그저 본인의 혼동이 있었을 뿐, 위 설정들이 다 맞으면 해결이 될 것이다. 만약 동일한 문제를 만났다면 위 조건 부터 맞춰보자.

특정 node에서만 실패한다면 그 node의 환경설정이 깨졌을 가능성이 큰데, 실패하는 node를 성공하는 node로 그대로 엎어치면 해결 될 것이다. 하지만 원인을 밝혀보자.

## 진행

현장에서 재현해보고 해결해 보자

1. 실패하는 node의 shell에 접속 후, 실패하는 build project의 path로 가서 동일하게 빌드 명령 수행
2. 동일한 에러 로그를 발생하며 실패
3. 위 분석에서 제시하는 설정 값들을 제대로 동작하는 node의 설정과 동일하게 맞추어 보기
4. 재시도 해보았지만 여전히 실패

## 해결책

위 문제를 곰곰히 생각해보면 한가지 확실한 명제가 있는데,

> "빌드 할 project와 Java의 버전을 일치(호환) 시켜라"
> 

이다. gradle.build에는 필요한 Java 버전이 명시되어 있고, 여기에 맞는 JDK가 설치되어 있고, 그 버전으로 빌드를 수행하게 하면 된다는 것이다.

Jenkins의 shell에 접속해서 동작하는 node와 위 조건들을 다 맞추고 직접 빌드를 해봐도 계속 동일한 에러가 뜬다. 그런데 혹시나 해서 Jenkins에서 다시 빌드를 했더니 이제 성공한다.

왜 shell에서 직접 수행할 때는 여전히 실패하지만 Jenkins에서는 다시 성공하는 것일까?

결론은 2가지 조건이 맞아야 한다는 것이다.

- Java 버전을 맞춰라 → 동작하는 node의 설정 값들과 동일하게 맞추면서 JDK를 새로 설치함
- JAVA_HOME을 맞춰라 → 여전히 JDK 8로 해뒀는데?

Shell에서는 JAVA_HOME을 JDK 13으로 바꾸지 않아서 직접 빌드는 실패했지만, Jenkins에서는 JAVA_HOME을 재설정하므로 빌드가 성공한 것이다.

Jenkins는 빌드 구성(build configuration)에 JDK를 지정하는 부분이 있다. 여기서 선택할 수 있는 JDK는 Jenkins의 "Jenkins관리(manage) > Global Tool Configuration(configureTools) > JDK (JDK installations)" 에 등록되어 있어야 한다.

여기 등록되는 JDK들은 JAVA_HOME을 빌드 과정에서 override하게 되는데, 여기 정의된 JAVA_HOME에 맞는 path에 JDK가 설치되어 있어야 하는 것이다.

## 결론

아마 이 문제는 secondary에서 주로 발생할 수 있을 것이다. Secondary에 적절한 JDK가 설치되지 않으면 Jenkins의 JAVA_HOME override시 인식이 되지 않아 기본 JDK가 실행 될 것이기 때문이다.

결론적으로 JAVA_HOME 설정에 적합한 Java 버전을 맞추는게 정답이지만, 앞서 IntelliJ의 문제와 마찬가지로 실제로 빌드를 수행하는 환경에서 쓰는 Java버전, JAVA_HOME을 맞추는 것도 중요하다.

## 참고

- Java 버전 바꾸는 법: [https://dev.to/harsvnc/how-to-change-your-java-and-javac-version-on-ubuntu-linux-1omj](https://dev.to/harsvnc/how-to-change-your-java-and-javac-version-on-ubuntu-linux-1omj)

- Java 버전별로 설치하기 위한 sdkman: [https://sdkman.io/install](https://sdkman.io/install)
