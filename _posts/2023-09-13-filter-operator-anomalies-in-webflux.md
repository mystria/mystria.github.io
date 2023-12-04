---
layout: post
title:  "WebFlux 에서 필터 연산자 변칙 활용"
date:   2023-09-13 19:00:00 +0900
categories: Java WebFlux Reactive Operator
comments: true
---

# Sequence 처리 도중에 흐름을 바꾸고 싶다

아래와 같은 요구사항이 있다.

> 1. User 를 순회하면서 체크인을 수행한다.
> 2. 중간에 한 명이 체크인을 실패할 경우, 더 이상 체크인을 수행하지 않는다.
> 3. 최종적으로 체크인을 했든 못했든, 모든 user 결과 목록을 반환한다.
> 

말로는 설명하기 힘들어서 코드로 예를 들어보자. 

```java
public List<User> checkInUsersImperative(List<User> users) {
    boolean circuitBreaker = false; // flag 변수
    List<User> checkedInUsers = new ArrayList<>();
    for (User user : users) {
        if (circuitBreaker) { // 데이터 순회 중 하나가 실패하면 이후 데이터는 강제로 실패 처리
            user.fail();
            checkedInUsers.add(user);
            continue;
        }
        User checkedInUser = this.checkIn(user);
        checkedInUsers.add(checkedInUser);
        if (UserState.FAILED.equals(checkedInUser.getState())) {
            circuitBreaker = true; // 실패 시 flag 변경
        }
    }
    return checkedInUsers;
}

private User checkIn(User user) {
    try {
        // 뭔가 비즈니스 로직
        ...
        return user.success();
    } catch (Exception e) {
        return user.fail();
    }
}
```

checkIn() 메서드가 성공하면 user 는 CHECKED_IN 상태가 되지만, 실패하면 FAILED 상태가 된다. 그리고 한명이 실패하면 더 이상 체크인을 하지 않고 나머지 user 들도 실패 처리한다.

명령형 코드에서는 익숙하게 구현할 수 있다. 외부에 상태를 관리할 flag 변수를 두면 되기 때문이다. 하지만 함수형으로 구현하려면? 외부에 flag 변수를 변경(side-effect)하여 중간에 상황을 변화시키는 것은 불가능한 일이다.

그러면 선언형이자 함수형인 WebFlux 로 이런저런 기능을 구현할 때는 어떻게 해야 할까? 

# WebFlux 의 연산자로 해결 불가

WebFlux 는 Reactive 연산자들을 제공한다. 여기서 특정 상황이 되면 동작을 바꿀 기능이 있는지 찾아보았지만 찾지 못했다. (혹시 있는데 못찾은 것 뿐 일지도…)

실패가 발생하면 error 가 발생하니 error 처리 연산자로 해결해보려고 했으나 실패했고, filter 연산자를 아무리 응용하여 짜 맞춰봐도 상태 flag 가 없이는 불가능해 보였다.

## WebFlux 의 error 처리 연산자들

Error 처리 연산자는 원하는 기능을 지원하지 않는다. onErrorContinue 는 이후 데이터까지 처리해 버린다.

| Error 처리 연산자 | 설명 |
| --- | --- |
| error() | Parameter 로 지정된 에러로 종료하는 mono/flux 생성 |
| onErrorComplete() | 에러 이벤트를 완료 이벤트로 변경 |
| onErrorReturn() | 에러 이벤트 대신 대체 값을 emit - sequence 종료 |
| onErrorResume() | 에러 이벤트 대신 대체 publisher 를 반환(에러 이벤트 확인 가능) - sequence 종료 |
| onErrorMap() | 에러 이벤트를 다른 에러로 변경하여 전파(onErrorResume() 사용) |
| onErrorContinue() | 에러 영역 내에 있는 데이터는 제거하고 후속 데이터는 emit(에러 이벤트와 값 확인 가능) |
| retry() | Parameter 로 입력한 횟수만큼 원본 sequence 를 (처음부터)재구독  |

## WebFlux 의 filter 연산자들

필터 연산자는 원하는 기능을 지원하지 않는다.

| Filter 연산자 | 설명 |
| --- | --- |
| filter() | Upstream 에서 emit 된 데이터 중 조건에 일치하는 데이터만 downstream 으로 emit |
| filterWhen() | Inner sequence 를 통해 비동기적으로 조건을 확인하여 일치하는 데이터만 emit |
| take() | Parameter 로 입력받은 숫자만큼만 데이터를 emit |
| takeLast() | Parameter 로 입력받은 숫자만큼 upstream 의 마지막 데이터를 downstream 으로 emit |
| takeUntil() | Predicate(Lambda 표현식)가 true 가 될 때(inclusive) 까지 데이터를 emit |
| takeWhile() | Predicate(Lambda 표현식)가 true 인 동안에만(exclusive) 까지 데이터를 emit |
| skip() | Parameter 로 입력받은 숫자만큼 건너뛰고 나머지 데이터를 emit |
| skipXXX() | takeLast(), takeUntil(), takeWhile() 과 비슷 |
| next() | 첫 번째 데이터만 emit |

# 그렇다면 다소 꼼수를 쓰자

- 특정 조건이 될 때까지 값을 수집하는 takeUntil()
- 두 publisher 를 합치는 zip()

위 두가지 연산자를 활용해보자.

하나의 publisher 는 특정 조건이 될 때까지 takeUntil() 하고, 또 하나의 publisher 는 원본 값을 발생시킨다. 그리고 이 둘을 zip() 하되, takeUntil 로 얻은 것을 우선 취하고, 이후에는 원본 값을 취한다.

이후 이를 다시 흘려보내면…

## 해결

예제 코드

```java
public Mono<List<User>> checkInUsers(List<User> users) {

    return Flux.fromIterable(users)
        .concatMap(this::checkIn) // Important : concatMap
        .takeUntil(user -> UserState.FAILED.equals(user.getState()))
        .collectList()
        .zipWith(Mono.just(users), this::fillSkippedUsers);
}

private List<User> fillSkippedUsers(List<User> checkedInUsers, List<User> users) {

    Map<String, User> checkedInUsersMap =
        checkedInUsers.stream().collect(Collectors.toMap(User::getId, Function.identity()));
    return users.stream()
        .map(user -> checkedInUsersMap.computeIfAbsent(user.getId(), user.fail()))
        .toList();
}

private Mono<User> checkIn(User user) {
    // 뭔가 비즈니스 로직
    ...
    .onErrorResume(throwable -> {
        log.error("Fail to check in, {}", throwable.getMessage(), throwable);
        return Mono.just(user.fail());
    });
}
```

만약 concatMap() 대신 flatMap() 을 사용할 경우, 다음 요소를 바로 실행시켜 버리므로 checkIn() 에서 지연이 걸린다면 일부 요소들은 추가로 체크인을 해버리게 된다. Non-blocking 의 철학을 고려해 보면 concatMap 이 좋은 방법은 아닌 것 같지만, 구현하려는 목표에 따라 적절히 사용하면 된다고 본다.

# 결론

애초에 위와 같은 상황이 발생한 것 자체가 설계의 문제일 수도 있다. 이 문제를 처음 맞닥드렸을 때 WebFlux 를 제대로 이해하고 있진 않았다(지금도 어렵다). WebFlux - Reactive 를 더 잘 이해했다면 이 상황에 빠지지 않을 수도 있었을까 싶다.

그러나 개발이란 게 좋은 설계 위에서만 할 수 있는 건 아니니까… 누군가에게 참고가 될 수 있지 않을까 희망한다.