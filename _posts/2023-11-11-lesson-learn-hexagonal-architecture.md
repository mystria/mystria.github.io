---
layout: post
title:  "Hexagonal Architecture 후기"
date:   2023-11-11 21:00:00 +0900
categories: Architecture
comments: true
---

# Hexagonal Architecture 는 정말 좋았을까?

[Hexagonal Architecture 정리]({% post_url 2023-02-26-summarize-hexagonal-architecture %}) 이 후, 실제 Hexagonal Architecture(이하 육각형 아키텍처)를 적용하여 프로젝트를 수행하며 배우고 느꼈던 점, 성과와 실패를 정리해 본다.

# 주의

어떤 기술이나 도구든 팀과 조직의 역량, 그리고 프로젝트 일정과 성향에 따라서 적합한지 확인해야 한다. 이 글은 개인의 경험에 기반한 내용일 뿐 육각형 아키텍처의 객관적 지표가 아님을 분명히 한다. 개인의 역량 부족, 잘못된 활용, 프로젝트 상황 및 다른 기술 스택의 영향 등이 있을 수 있다. 또한, 여기서 작성된 장단점은 육각형 아키텍처”만” 가진 특징이 아님을 짚고 넘어간다.

해당 프로젝트는 계속 진행 중(2023년 11월 기준)이므로 향후 의견이 추가되거나 바뀔 수 있다.

> 도메인 로직, 비즈니스 로직 - 육각형 아키텍처에서는 도메인 계층이 존재한다. 여기에 구현되어야 하는 로직, 결국 가치를 일으키는(돈을 벌어다 주는) 로직에 대해서 비즈니스 로직으로 통칭한다.
> 

# 좋았던 점

- 의존성으로부터의 독립 그리고 테스트
    - 육각형 아키텍처를 유지하기 위해 의식적으로 의존성 분리, 가급적 서버 컨테이너 설정도 분리하려고 노력함
    - 의존성이 분리되니 테스트가 너무 간편해짐 - 인터페이스는 mocking 하고 요청과 그 결과를 검증하는 반복적인 단계
    - 테스트가 쉬우니 테스트 코드 작성도 빨라지고 많아짐, TDD 적용 가능
- 프로젝트의 모듈화
    - 프로젝트의 각 계층을 모듈로 분리할 수 있음, 즉, 모듈별로 업무를 나누어 R&R 관리 쉬움
    - 기능 또는 업무를 잘게 쪼갤 수 있음 - 빠른 커밋과 피드백 가능
- 비즈니스 로직에 집중
    - 도메인 계층은 비즈니스만 고민 - 기획자와 함께 고민 가능(DDD…)
    - 코드 구조나 라이브러리, 리소스를 어떻게 쓸지 고민하기 보다 요구사항을 어떻게 구현할지에 대해 집중
- 외부 서비스와 연계를 최대한 지연시킬 수 있음
    - 연동해야 할 외부 서비스가 아직 준비되지 않았다면 이 부분을 dummy 로 대체하여 우리 비즈니스 로직부터 개발할 수 있음, 느슨한 연결
    - 나중에 외부 서비스가 준비되면(또는 변경되면) 연동 부분에서만 대응하면 됨
- 완성되어 가면서 느끼는 안정감
    - 지극히 단순한 구조 유지 가능, 복잡한 부분도 결국은 모듈 단위로 추상화되어 격리됨
    - 그냥 만들었다면 기능에 따라 각기 다른 구조(structure)가 나오거나 변경될 수 있지만, 육각형 아키텍처의 철학에 맞추면서 지속해서 단순하고 일관된 구조를 지향하게 됨
    - 테스트가 많이 만들어져 신뢰감 생성
- 나쁘지 않은 구조를 강제
    - 육각형 아키텍처보다 효율적인 구조였더라도 라이프사이클이 진행되며 점점 나빠지다 최악으로 빠지는 경우 존재, 이를 사전 차단
    - 구조를 유지할 아키텍처의 기준이 없으면 그때그때 필요한 모양으로 변하고 차후에는 일관성 없고 가독성이 떨어지는 구조로 변하게 되어 기술 부채가 되어버림, 특히, 외부 연동 또는 DB entity 에 의해 비즈니스 로직이 흔들려 버릴 수 있음
    - 여러 기능을 구현하더라도 일정한 틀을 유지해 주므로 시간이 흐르더라도 깨끗한 구조가 유지될 가능성이 높음

# 나빴던 점

- 불안하고 (심리적으로) 어려움
    - 왜 해야 하냐에 대한 물음에 대쪽과 같이 응대할 수 없음 - 이론적으로는 말할 수 있겠지만, 주변에 활용 사례와 레퍼런스를 찾기 어려움(아키텍처의 대중화 필요?)
    - 스스로도 할 수 없을 것 같은, 괜히 한 것 같은 의구심이 끊이지 않음
    - 하고 나서도 뭔가 찜찜함 - 성공 사례와 비교할 수 없음
    - 상급자와 동료들의 지지가 필요함
- 프레임워크로부터의 완벽한 독립은 어렵거나 불가능
    - 비즈니스 로직과 무관하지만 비즈니스 로직에 포함되어야 하는 애매한 데이터들(예를 들면 tracking id 등)을 어떻게 처리할지 의사결정 필요
    - 트랜잭션이나 컨텍스트 데이터 처리는 프레임워크를 활용할 수 밖에 없음 - 비즈니스 로직에 집중하려던 철학에 금이 가기 시작
- 비즈니스 로직 분리나 성능 개선 지점의 모호함으로 고민거리가 많아짐
    - 추상화된 인터페이스의 구현체는 비즈니스 로직에서 최적화를 할 수 없음
    - 예를 들어 데이터의 종류에 따라 세밀한 처리가 필요할 때, 인터페이스를 성능적으로 세분화하는 것과 논리적으로 추상화 할지에 대한 의사결정 필요
    - 트랜잭션의 범위를 잡기 어려움, 글로벌 트랜잭션에 대한 수요가 생길 수 있음
    - 도메인 영역에서 어디까지 알아도 되고 몰라야하는지 판단이 필요
- 중복 코드 많아짐
    - 모듈 단위에서 DTO 변환이 잦음
    - 포트와 어댑터로 분리된 모듈 사이에서 거의 동일한 멤버 프로퍼티를 갖는 DTO 간의 변환이 요구됨, 반복되는 코드로 인해 가독성이 저하됨
    - 코딩 작업 시 왔다 갔다 많이 해야 함
- 정말로 독립할 수 있을까?
    - 어댑터들의 데이터 변경이 도메인에 영향을 미칠 수 밖에 없음
    - 데이터 타입이 변하거나 변수명이 변경되는 것 정도는 문제가 없으나, 중요 데이터가 추가/삭제 될 때에는 영향을 받을 수 밖에 없음
    - 도메인에 대한 강한 확신과 이를 유지하기 위한 로드맵이 필수, 이것이 없다면 변경되는 코드양이 엄청 많아짐
- 프로젝트가 방대해짐
    - 별 기능이 없는데도 규모가 커짐, 간단하게 유지하고 싶지만 기본적으로 크기가 큼
    - 거대한 모노리스를 만든 것 같은 느낌
    - 그냥 서비스를 잘게 쪼개서 MSA 로 만든 것 보다 뭐가 더 좋지?

# 생각과 고민들

- 역시 만능키는 아님
- 개발 자원(인력, 시간)을 상당히 많이 요구함 - 익숙해지면 괜찮음, 네이밍 룰이나 컨벤션이 중요!
- 적고 보니 좋은 점 보다 나쁜 점이 많아 보임 - 그럼에도 해볼 만 하다
- 단점을 수용하고 제어할 수 있으면 더 이상 단점이 아님, 어려울 뿐 나빠지는 것이 아님
- 굳이 육각형이 아니라 그냥 계층형 아키텍처에서도 테스트를 충분히 잘 구축하고 비즈니스 로직을 잘 분리할 수 있음, 그러나 한 번 육각형을 경험한 뒤에는 계층형도 자연스레 육각형을 따라갈 것 같음
- 앞으로도 계속 쓸 것이냐 묻는다면?
    - 상술한 장단점과 프로젝트의 특성을 반드시 검토 - Trade-off 를 고려!
    - 동료들과 협의만 된다면 계속 쓰고 싶음

# 추천 대상

- 기술적인 해결책보다 비즈니스 로직이 훨씬 더 중요하고 복잡할 경우
- 비즈니스 요구사항에 대한 확신이 있을 경우
- 큰 프로젝트
- 개발 조직이 충분히 커서 리소스를 나누거나 여유있게 개발할 수 있을 경우