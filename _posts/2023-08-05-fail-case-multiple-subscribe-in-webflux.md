---
layout: post
title:  "오늘의 실패사례 - WebFlux 여러번 구독 문제"
date:   2023-08-05 11:00:00 +0900
categories: Java WebFlux Reactive
comments: true
---

# WebFlux 를 실행하며 만난 에러

컴파일도 잘 되었고, 테스트도 잘 통과 했는데, 실제 서비스를 실행하면 에러가 발생한다. 에러 메시지는 낯설고 뭐가 문제인지 모르겠다.

```
Rejecting additional inbound receiver. State=[terminated=false, cancelled=false, pending=0, error=false]
```

## 원인

결국 외부 서비스의 응답을 사용하는 과정에서 “또” 조회를 하는 바람에 발생한 에러였다.

WebFlux 로 개발 할 때, 테스트 및 자체적으로 생성하는 Mono(혹은 Flux) 는 계속 생성되기 때문에 고민없이 구현하다보면 잘 된다고 생각한 부분에서 오류를 만나게 된다.

예를 들어 아래와 같은 코드는 그냥 잘 된다. 코드에 정의된 Flux 는 계속 생성될 수 있기 때문이다.

```java
public void someMethod() {
    List<User> users = List.of( ... some random generated users ... );
    Flux<User> userFlux = Flux.fromIterable(users);

    userFlux.map( ... )
        .subscribe(System.out::println);
    userFlux.map( ... )
        .subscribe(System.out::println);
}
```

하지만 다음과 같은 코드는 에러가 발생한다. Upstream 을 다시 생성시킬수 없기 때문이다.

```java
public void someMethod(Flux userFlux) {
    userFlux.map( ... )
        .subscribe(System.out::println);
    userFlux.map( ... )
        .subscribe(System.out::println);
}
```

실제 서비스에서는 source 는 단 한 번 만 생성(publish)되는 경우가 많을텐데, 이를 습관적으로 보통의 Java 객체처럼 생각하고 여러번 구독하면 에러가 발생한다. 

## 결론

Mono(Flux) 는 한 번 구독하면 소모되어버린다는 점을 잊지 말자.

# Cache 로 해결

How to use "cache" method of Mono

- [https://stackoverflow.com/a/67487325/8350542](https://stackoverflow.com/a/67487325/8350542)
- [https://stackoverflow.com/a/52518786/8350542](https://stackoverflow.com/a/52518786/8350542)

```java
var cached = mono.cache();
```

## Cache 에 대한 설명

요즘은 ChatGPT 에서 잘 설명해 준다.

> `Mono`를 두 번 구독할 수 없는 제약은 리액티브 스트림의 원칙에 따른 것입니다. 리액티브 프로그래밍에서는 데이터의 비동기적인 흐름을 제어하기 위해 이러한 제약이 적용됩니다. 만약 동일한 데이터를 두 번 이상 사용해야 한다면 다음과 같은 방법을 고려할 수 있습니다:
>
> 1. **`cache()` 사용**: `Mono`의 `cache()` 연산자를 사용하여 데이터를 캐시하고 여러 번 구독할 수 있도록 할 수 있습니다. `cache()` 연산자는 `Mono`를 구독할 때 데이터를 캐시하여 나중에 다시 사용할 수 있도록 합니다. 단, 데이터가 크거나 지속적으로 변경되는 경우 캐시 사용에 주의해야 합니다.
> 

참고로 cache() 도 구독이다. cache() 하면서 source 를 읽어왔기 때문이다.