---
layout: post
title:  "금주의 실패사례 - Java13 에서 11로 내렸더니 compile 실패"
date:   2021-11-04 04:00:00 +0900
categories: Java
comments: true
---

# Java 13 에서 11로 내렸더니 compile 실패
살다 보면 내리막길도 있다.  
Java 13로 실컷 개발해왔는데 어른들의 사정으로 11로 내려야 하는 경우도 있는 것이다.  
알 수 없는 오류를 만난 건, Gradle 설정과 JDK 환경변수 설정까지 끝내고 gradle build를 하는 순간이었다.

## 증상
* 증상 1

	```
	> Task :compileJava
	Note: /src/main/java/your/project/MenuCacheBuilder.java uses unchecked or unsafe operations.
	Note: Recompile with -Xlint:unchecked for details.
	compiler message file broken: key=compiler.misc.msg.bug arguments=11, {1}, {2}, {3}, {4}, {5}, {6}, {7}
	java.lang.AssertionError
		at jdk.compiler/com.sun.tools.javac.util.Assert.error(Assert.java:155)
		at jdk.compiler/com.sun.tools.javac.util.Assert.check(Assert.java:46)
	```

	위와 같은 에러... 에러 메시지 조차 에러가 남

* 증상 2
	```
	> Task :compileJava
	compiler message file broken: key=compiler.misc.msg.bug arguments=11, {1}, {2}, {3}, {4}, {5}, {6}, {7}
	java.lang.AssertionError
		at jdk.compiler/com.sun.tools.javac.util.Assert.error(Assert.java:155)
		at jdk.compiler/com.sun.tools.javac.util.Assert.check(Assert.java:46)
	```

	이건 Note 도 없음

## 분석
* 첫 번째는 그래도 Note로 compile 중인 class를 보여주기 때문에 해당 class에 들어가서 SonarLint로 warning 확인

	> Provide the parametrized type for this generic.
	> 

	이는 generic type에 type 선언을 제대로 해주지 않아서 발생(예: List<String>을 List만 쓴 경우) 한 것으로 고쳐주면 해결됨

* 두 번째는 이유를 도무지 알 수 없음
  + Assert.check() method에 breakpoint를 찍고 build를 debug 해봤더니 왠 instance method reference에서 compile 에러 발생
  + User user에게 isActive() 이라는 method가 있을 때,  
  .filter(user → user.isActive()) 를 .filter(user::isActive) 로 method reference로 표현한 경우는 무사히 compile 됨
  + Company company에게 isMember(User user) 라는 method가 있을 때,  
  .filter(user → company.isMember(user)) 를 .filter(company::isMember) 와 같이 표현한 경우는 compile 실패, 

* 즉, 특정 method reference를 지원하지 못함
* 이 부분은 SonarLint에서도 method reference 화 하라고 추천하는 부분이며 Java 13에서는 문제없이 compile 잘 됨
* 아마 위 기능이 Java 13부터 지원하거나 버그가 해결된 것으로 추측

## 기타
> compiler message file broken: key=compiler.misc.msg.bug arguments=11, {1}, {2}, {3}, {4}, {5}, {6}, {7}
> 

이 부분은 메시지도 못 만들어내는 부분으로 Java 버전들마다 있는 버그로 보임