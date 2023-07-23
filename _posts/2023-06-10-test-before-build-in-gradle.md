---
layout: post
title:  "Gradle 로 테스트 후 빌드 하기"
date:   2023-06-10 13:00:00 +0900
categories: Java Gradle IntelliJ Test
comments: true
---

# 느린 테스트는 빼고 테스트 후 빌드 하기

아무튼 우리는 테스트를 해야한다. 그리고 빌드 시에 테스트도 함께 수행하도록 하면 나의 작업이 regression 되지 않았는지 주의하며 진행할 수 있다. 다만, 빌드 시 느린 테스트는 빼고 빌드를 하게 만들고 싶었다.

그러나 IntelliJ 에서 다소 사소한 문제를 만나게 되는데…

## 배경

시간이 오래 걸리는 Slow Test 를 그냥 수행하고 빌드하는 방법도 있을 수 있다. 그러나 Integration Test 같은 경우에는 빌드 환경에서는 동작하지 않을 수도 있고, 불필요하게 많은 설정을 요구할 수도 있다. 꼭 느리지 않더라도 로컬 개발에서만 테스트해보고 빌드 파이프라인에서는 무시하고 싶은 테스트가 있을 수도 있다.

## 기본적인 방법

Gradle 에서는 테스트를 어떻게 수행할 지 정의할 수 있다. 여기에서 특정 테스트를 제외하는 방법은 이미 많이 공유되고 있으니 쉽게 찾을 수 있다.

### Tag, Cateogry 를 이용해 특정 테스트를 포함/비포함

JUnit 5 부터는 @Tag 또는 @Category 라는 annotation 을 지원하여 테스트를 태깅할 수 있으며, 이를 이용해 테스트에 포함하거나 제외 할 수 있다.

[https://www.baeldung.com/junit-5-gradle#configuring-junit-5-tests-with-gradle](https://www.baeldung.com/junit-5-gradle#configuring-junit-5-tests-with-gradle)

```groovy
test {
    useJUnitPlatform {
    	  includeTags 'fast'
        excludeTags 'slow'
    }
}
```

### Package 나 파일명 wildcard 를 이용하여 포함/비포함

뿐만 아니라 Gradle 에서는 filter 를 통해 특정 패키지에 포함된 테스트나, 특정 이름의 테스트 클래스만 수행되게 할 수 있다.

[https://docs.gradle.org/current/userguide/java_testing.html#test_filtering](https://docs.gradle.org/current/userguide/java_testing.html#test_filtering)

```groovy
test {
    filter {
        //include specific method in any of the tests
        includeTestsMatching "*UiCheck"

        //include all tests from package
        includeTestsMatching "org.gradle.internal.*"

        //include all integration tests
        includeTestsMatching "*IntegTest"
    }
}
```

### Gradle 명령시 특정 테스트 포함/비포함 옵션 전달

만약 gradle 빌드 명령을 지정할 수 있다면, —tests command 를 통해 특정 테스트를 지정할 수 있다.

[https://stackoverflow.com/questions/22505533/how-to-run-only-one-unit-test-class-using-gradle](https://stackoverflow.com/questions/22505533/how-to-run-only-one-unit-test-class-using-gradle)

```bash
$ gradle test --tests org.gradle.SomeTest.someSpecificFeature
$ gradle test --tests '*SomeTest.someSpecificFeature'
$ gradle test --tests '*SomeSpecificTest'
```

## 기본적인 방법의 문제점

보통은 위 방법들을 사용하는 것 만으로도 충분하다.

> 주의: 개발자 환경마다 다를 수 있음
> 

하지만 문제가 되는 지점은 우리가 IntelliJ 등 IDE 를 통해 테스트를 수행할 때 발생한다. Gradle 프로젝트는 기본적으로 테스트를 gradle 의 task 로 실행하게 되는데, 이때 gradle 설정을 참조하게 된다.

만약 위 방법들을 통해서 테스트를 gradle 설정에서 제외시키면, 제외된 테스트는 (당연하게도) gradle 의 task 로 실행할 수 없게된다. 제외된 테스트를 직접 특정하여 시도해보면 테스트가 존재 하지 않는다고 표시된다.

```
No tests found for given includes: [com.mys.project.YourSlowTest](--tests filter)
```

물론 직접 gradle 명령을 통한 방법이나, JUnit 으로 테스트를 수행할 때에는 gradle 설정에서 제외 되었더라도 문제 없이 실행된다. 본문은 gradle 을 이용한 테스트에 대해 한정한다.

# 기본적으로는 제외하되 직접 실행은 되게 하기

IDE 에서 gradle 을 통해 실행되는 테스트는 gradle 설정의 포함/비포함의 영향을 받게 된다. 그래서 제외된 테스트는 gradle 로는 더 이상 실행할 수 없게 되었다.

그렇지만 별도로 특정해서 테스트를 수행하고 싶을 때는 제외되지 않게 하고 싶다면?

## 방법

일단 이런 문제가 다른 사람들에게는 큰 문제가 아닌듯 잘 검색되지 않는다.

오랜시간 검색 끝에 다음의 질문과 답변이 큰 힌트가 되었다.

[https://stackoverflow.com/questions/71828925/how-to-run-a-single-test-that-is-excluded](https://stackoverflow.com/questions/71828925/how-to-run-a-single-test-that-is-excluded)

> commandLineIncludePatterns 이 존재하지 않으면 제외시켜라
> 

IDE 에서 gradle 로 특정 테스트를 실행하면 IDE 는 자동으로 다음과 같은 명령어를 만들어 실행 시킨다.

```
$ gradle :module:test --tests "com.mys.project.YourSlowTest"
```

특정한 테스트만 실행시키기 위해 —tests 이후에 내가 특정한 테스트 "com.mys.project.YourSlowTest" 가 command 로 전달이 되는데, 이 부분이 바로 `commandLineIncludePatterns` 이다.

반면 빌드를 했을 때 수행되는 테스트에서는 —tests 를 전달하지 않는다.

참고로 아래는 Gradle 팀의 `commandLineIncludePatterns` 에 대한 내용.

[https://discuss.gradle.org/t/get-tests-specified-by-tests-commandline-argument/40324/9](https://discuss.gradle.org/t/get-tests-specified-by-tests-commandline-argument/40324/9)

## 해결

빌드 시에는 느린 테스트를 제외시킨다. 느린 테스트에는 tag 를 붙여둔다. 단, 느린 테스트를 단독 수행시에는 테스트에서 제외 시키지 않는다.

commandLineIncludePatterns 에 대한 조건은 doFirst 로 정의해야 한다. 그래야 테스트가 시작되기 전에 excludeTags 가 적용될 수 있기 때문이다.

```groovy
test {
    useJUnitPlatform()
    doFirst {
        // build 시에는 slow-test 를 제외하고 test 를 수행한다.
        // 그러나 개별 test 수행 시에는 slow-test 도 수행할 수 있게 한다.
        if (filter.commandLineIncludePatterns.empty) {
            useJUnitPlatform {
                excludeTags "slow-test"
            }
        }
    }
}
```

# 결론

IntelliJ 환경에서 Gradle 프로젝트의 특정 테스트만 잘(?) 제외시킬 방법에 대해 찾아보았다.

많은 개발자에게 도움이 될만한 내용은 아니지만, 비슷한 문제를 해결하는데 도움이 되길 바라며…