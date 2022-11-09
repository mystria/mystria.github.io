---
layout: post
title:  "당근마켓의 Kotlin"
date:   2021-12-20 16:00:00 +0900
categories: Kotlin
comments: true
---

# 당근마켓 Server MeetUp 후기
2021년 12월 20일 Youtube에서 진행된 당근마켓 Server MeetUp을 개인적으로 정리한 글입니다.

- 당근마켓은 MSA로 여러 컴포넌트가 있는데, 일부는 Node.js, 일부는 Kotlin을 쓰고 있음
- Kotlin으로 통일하기로 결정하고 Kotlin + Spring으로 체질 변경
- 그리고 Kotlin의 장점 대 방출

## 왜 Kotlin을 선택했을까?

특정 언어의 우월성을 이야기 하는 것이 아님

- MSA 구성, 데이터 관리에 집중 (성능, 생산성 등은 둘다 괜찮다)
- Node.js는 gRPC를 위한 공식 모듈이 없어 직접 구현 필요
- Kotlin의 gRPC의 호환이 좀 더 성숙 (JVM 계열의 장점)
- Kotlin의 아래 장점들
    - 인터프리터 vs 컴파일
    - 비동기(성능? 글쎄..) vs 동기(그러나 비동기 기능 지원)
    - 패턴매칭 지원
    - 조건절 반환값 저장
    - 다양한 컬랙션 API (List가 강력)

## 왜 언어를 통합하기로 했을까?

MSA, 다른 언어로 된 MSA를 운영하며 “같은 고민의 반복”, “노하우의 파편화”, “중복로직”, “팀간의 소통, 채용 문제” 등으로 하나의 언어로 통일하기로 함

잘 돌아가고 있는 상황에서 전환할 방법

1. 한 가지 언어로 옮기기 : 버그, 누락, 실수 재현할 수 있지만 간단하고 확실함
2. 그냥 현재처럼 쓰기 : 어차피 분리된 컴포넌트, 차라리 신규 개발에 집중하는게 낫고 자율성 및 확장성 높음
3. Hybrid : 일단 그냥 두고 문제 해결이 필요할 때 차차 갈아타기 (이 부분은 확실히 못들음)

1번을 선택 후 3개월 동안 빠르게 진행!

## Kotlin의 (언어적) 장점

Google의 공식 언어 중 하나가 되고, Spring 5의 공식 지원(Spring 개발자들의 Kotlin에 대한 많은 노력)

- Kotlin 특징
    - Java와 상호운영성에 초점 (Java 와 100% 호환, IntelliJ 팀이 도구를 만들기 위해 만든 언어)
    - 간결 & 안전
    - 간결성
        - 보통 코드 작성 보다 읽는데 시간이 더 많이 소요, 코드는 간결할수록 쉽고 의도 전달이 잘 됨
        간결 == 작성시간 ↓, 읽는시간 ↓ == 개발속도↑
        - 처음부터 부가적(실제 유의미한 로직이 아닌) 요소를 적게하기 위해 만들어진 언어, getter/setter/equals 같은 건 자동으로 지원(No Lombok)
        - 풍부한 표준 라이브러리, 함수형(function type), [syntax sugar](https://dkje.github.io/2020/09/02/SyntaxSugar/) 제공
    - 안정성
        - 오류를 막기위한 방어코드 → 더 많은 방어코드 → 부가적인 요소가 많아짐 → 생산성 하락 (안정성과 생산성은 trade off)
        - Null 금지, 자동 casting, 열거형 평가 등을 컴파일 단계에서 지원
- Kotlin의 활용
    - Null을 잊자
        - “?” 표현, ? 변수 뒤에 붙어있다면 null 일 수 있다는 뜻, 즉 검증 코드 필요
        - Elvis 연산자, ?: 의 왼쪽이 null이면 null 반환, 아니면 오른쪽 값 반환
        - Safe call, ?. 의 왼쪽이 null이면 null 반환, 아니면 .이후의 property/method 반환
        - “!!” 표현, null 이 아님을 단언
    - 문(statement)과 식(expression)
        - Statement는 값을 생성하지 않음, expression은 값을 생성하여 반환
        - Java는 모든 문이 문이지만, Kotlin은 문을 식으로 표현 가능(if문을 반환하면 그 평가 결과를 반환 ⇒ 제어’식’)
        - 함수의 반환값으로 if식을 보내 간단하게 처리하는 등의 방법
        - when() 역시 즉시 반환 가능, 패턴매칭의 타입체크(Java 17부터 지원)
        - sealed class: class 외부에 자신을 상속한 class 생성 불가, 반드시 sealed class 내부에 정의하도록 하여 불필요한 체크로직을 사전에 방지 (위반시 컴파일 에러 발생)
    - 오버로딩 불필요
        - Ref.to()로 인자 위치 변경 가능, 인자 이름으로 순서 무시 가능
        - 기본 인자를 지원하여, 빼고 전달하면 기본값으로 인식 = 오버로딩 불필요
        - 즉, builder pattern 을 언어레벨에서 지원하여 더이상 필요없어짐
    - Template callback pattern, Lambda 지원이후 잘 안쓰이는 패턴, 익명 클래스, 익명 객체
        - 함수형 인터페이스, fun interface로 정의
        - Kotlin은 함수가 [일급시민](https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4), 함수를 주고 받을 수 있음
        - 함수형(함수 타입), 하나의 기능을 가진 함수를 만들어 전달
    - Coding convention
        - 공식 coding convention을 제안, 유지보수성 ↑
- 풍부한 표현력
    - Scope 함수 (apply 등): 용례에 맞게 잘 쓰자
    - Kotlin + Spring: 환경변수에서 값을 꺼내는 방법 효율화, Kotlin 스러운 [확장함수](https://codechacha.com/ko/kotlin-extension-functions/) 제공
    - 이를 통해 조금 더 목적이 뚜렷한 코드 작성 가능
    - 테스트 시 @TestConstructor 제공: WebTest 시 MockMVC를 많이 쓰는데, DSL을 쓰면 간결해짐
        - DSL(Domain Specific Language), Java에서 DSL을 사용하려면 string이나 xml 파일로 관리 필요
        - Kotlin에서는 이를 좀 더 코드스럽게 관리 가능
        - MockK 로 Mockito 대신
- 함수형 스타일을 품은 Spring 5
    - (이 부분은 급하게 진행되어 잘 이해 못함)
    - Spring의 annotation 도입(2007년), 쉽게 기능을 도입 가능해짐
    - Java 8 이후 함수형 API로 새로운 시도, 경량 함수형 프로그래밍 모델
    - 기존 @Controller, @GetMapping 같이 구현한 부분을 route() 로 간결하게 해결
    - Bean도 함수형(Lambda식)으로 또는 DSL로 등록 가능
    - Spring Boot 는 annotation의 자동화 구성으로 도입이 어려웠음
    - Spring Fu 발표, Spring을 DSL로 작성 - JaFu, KoFu, 명시적 구성
        - annotation과 reflect 사용을 최소화 하여 속도 40% 향상
- 비동기 프로세스
    - Spring은 3.2 부터 지원, Mono나 Flux
    - Kotlin은 Coroutine을 통해 동기와 같은 구현 형태로 비동기를 지원
    - 비동기: 간단히 말해 동시에 수행하고 모두 다 끝나면 반환(하나씩 차례로 수행하는 것 보다 성능 향상)
        - Java의 CompletableFuture
        - Reactor의 reactive 라이브러리
        - Coroutine: 특별한 type 없이 suspend, async 예약어를 사용
    - 일반적인 비동기 코드로 구현 시
        - Java의 수많은 비동기 라이브러리... 복잡 다양
        - 콜백 지옥 같은 구현 - 가독성 저하
        - 동시성 처리, 에러 핸들링의 어려움
    - Kotlin의 비동기 처리 방법
        - 어떻게? 경량쓰레드? 일시 정지했다 callback 오면 다시 진행?
        - async로 표시해두면 컴파일 단계에서 구현 변환
        - Finite State Machine(FSM)을 사용한 동시성 처리, FSM 구현 형태로 변환
            - suspend 키워드가 붙은 함수로 구성
            - async 내의 suspend 키워드가 붙은 함수를 하나의 state로 만들어 state를 전이하게끔 recursive 호출
            - 마지막 state로 전이가 되면 결과 반환
            - 코드 레벨에서는 일반 동기식 코드와 모양이 동일
        - Coroutine이 해결사, 비동기 코딩이 필요할 경우 강력 추천
