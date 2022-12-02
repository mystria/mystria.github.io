---
layout: post
title:  "원격 개발 Gitpod 사용해 보기"
date:   2022-12-02 22:00:00 +0900
categories: Gitpod Codespaces JetBrains Tools
comments: true
---

# Gitpod 사용해 보기
GitHub 구경 중 발견한 Gitpod, 이게 뭘까?  
이런 저런 이유로 [Armeria 예제](https://github.com/line/armeria-examples)를 둘러보다가 `Open in Gitpod` 이라는 문구를 발견, 심심해서 눌러보았다.  
뭔가 가입하고 설치해야하는 것 같은데, 예제를 원격 개발 환경에서 직접 실행(및 수정)해 볼 수 있는 도구인가?  

## CDE
IDE 가 Integrated Development Environment 라면 CDE 는 Cloud Development Environment 이다. 기존에 조금씩 두각을 드러내던 클라우드 기반 원격 개발 도구(AWS Cloud9, 2017)들을 최근 원격 개발 관련한 요구가 많아지면서 다양한 클라우드 밴더에서도(Google Cloud Workstation, 2022) 출시하고 있다. Microsoft 도 GitHub 과 VS Code 를 앞세워 GitHub Codespaces 를 출시(2022)하였는데, Gitpod 도 이러한 도구들 중 하나이다.

Gitpod 는 2020년에 출시되었으며, '즉시 개발 가능한' 환경을 자동화하는 오픈소스 플랫폼이라고 한다.

> Gitpod is an open source developer platform automating the provisioning of ready-to-code developer environments.
>

마침 Gitpod 도 [[여기]](https://www.gitpod.io/vs/github-codespaces)에서 Codespace 와 기능을 비교하고 있다. 한국에서는 GitHub Codespaces 가 가장 유명한 것 같은데, Codespaces 의 [한글 소개](https://blog.outsider.ne.kr/1497)를 참조해보면 Gitpod 의 특성을 이해하기 쉬울 것이다. 한글로 된 Codespaces 관련 글은 충분히 많이 있다.

이런 원격 개발 도구들은 Pair programming, BYOD(Bring Your Own Device), IaC(Infrastructure as Code), Container 같은 기존 트랜드를 기반으로 재택 근무의 대두와 개발자 부족 현상까지 겹쳐 새로운 패러다임이 되고 있는 것 같다.

이런 클라우드 기반 원격 개발 도구들의 장점을 꼽으라면 다음과 같을 것이다.

- 개발자의 빠른 온보딩 - 환경 구축 시간 절감
- 코드 자산의 외부 유출 방지 - 로컬에 사본 없음
- 일관된 개발 환경 관리 및 유지 - 개발 환경의 코드화
- 언제 어디서나 개발 환경 접근 가능 - 원활한 재택/해외 개발자 고용
- 함께 개발 - 페어 프로그래밍, 코드 리뷰, 디버깅에 활용

## Gitpod 사용
그렇다면 Gitpod 는 어떻게 사용하는 것일까?  
기본적으로 Gitpod 은 Docker 를 사용한다. Workspace 요청 시 git repository (GitHub, BitBucket, GitLab 등)를 지정하면 Docker 를 이용해 개발 환경을 생성(provisioning)하여 endpoint 를 제공한다.  
JetBrains 계열과 VS Code 등의 IDE 나 에디터로 접속하면 원격 개발을 할 수 있다. 원격에서 코드 수정 뿐 아니라 실행, 디버그, 테스트, 코드 푸시, 심지어 터미널 작업 등 웬만한 작업은 다 할 수 있다.  
이 때 컨테이너를 사용하게 되고, 이 컨테이너가 유지되어야 하므로 비용이 발생하는 점도 유의해야 한다. 일단 개인 사용자는 50시간이 무료다.  
Git 을 Pod 로 만들기 때문에 Gitpod 인 것일까?  

## JetBrains Gateway
서두에 말했던 Armeria 예제를 Gitpod 으로 실행했더니 "IntelliJ IDEA 에서 여세요" 라는 메시지를 띄워줬다. 그래서 IntelliJ 에서 개발이 된다고? 싶었지만, Gateway 라는 프로그램을 설치해야 한다고 하며, 설치하더라도 JetBrains Client 라고 하는 경량 에디터가 실행되면서 개발 환경이 나타난다.  
최근 IntelliJ 의 UX 가 VS Code 와 거의 흡사해졌는데, 그래서인지 Client 도 VS Code 와 UX 가 비슷하다.  
참고로 이 Client 는 바로 [Code with Me]({% post_url 2022-12-02-jetbrains-code-with-me %}) 에서 언급한 원격 클라이언트 이다. 확신할 수는 없지만, 어떤 지점에서 원격 개발을 위한 프로토콜이 완성되어가고 있는 것으로 생각된다.

## 정리
Gitpod 를 이용하는 절차를 정리하면 다음과 같다.
1. 가입
    - 개인 개발자의 경우 무료로 써 볼 수 있음
1. 워크스페이스 생성
    - `https://gitpod.io/#project=프로젝트이름/https://github.com/프로젝트URL` 와 같은 형식의 링크
    - 또는, JetBrains Gateway 에서 repository 지정 후 `New Workspace` 생성
    - `.gitpod.yml` 을 이용한 개발 환경 설정 파일 구성
1. 워크스페이스 이미지 생성 및 배포 - endpoint 생성
    - 약 1분 미만의 시간 소요
1. 원격 개발 환경 접속
    - 별도의 개발용 client (JetBrains Gateway, VS Code, SSH 등) 사용
    - 여러명 동시 접속, 동시 편집 가능 (실제 사용은 안해 봄)
1. 개발 종료 시 워크스페이스 자원 반납
    - 사용 시간 만큼 과금

## 사용 소감
- 예제를 실행해 보기위해 로컬에 환경 구축(JVM, 컴파일러, SDK 등 설치) 하고 코드 clone 받는 작업이 사실상 진입 장벽인데, 이를 해결해 줌
- 그런데 이를 위해 설치해야 하는 도구들이 좀 있을 수 있음
- 익숙하지 않은 사람은 이게 뭔 상황이야? 할 수 있음
- 무료여서인지 Cold start 시간이 좀 긴 것 같음 - 하지만 SaaS를 상시로 띄워두면 비싼 것 같은데? 온프레미스(Self-hosted) 모델도 있음
- 이제 개발자가 맥북 프로를 쓸 필요 없이 크롬북🤭으로 원격의 맥북 프로급 가상 머신 위 개발 환경에 접속만 하면 되는 시대가 올 것 같음(마치 Steam Remote Play 나 GeForce NOW 같음)
- 하지만 네트워크 지연이 좀 거슬림 - 한국에 호스팅 하고 기가급 네트워크 망이 연결되면 해결될 지도... (하지만 VPN 이 등장하면 어떨까?!)

## 참고
- Gitpod 공식 문서 : https://www.gitpod.io/docs/introduction
- 구글 워크스테이션 : https://cloud.google.com/workstations
- AWS Cloud 9 : https://aws.amazon.com/ko/cloud9/
- GitHub Codespaces : https://github.com/features/codespaces
- JetBrains Space : https://www.jetbrains.com/space/