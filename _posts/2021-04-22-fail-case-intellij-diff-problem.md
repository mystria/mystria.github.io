---
layout: post
title:  "금주의 실패사례 - IntelliJ에서 Show Diff가 dialog로 안 뜰 때"
date:   2021-04-22 23:00:00 +0900
categories: IntelliJ
comments: true
---

# IntelliJ에서 Show Diff가 dialog로 안 뜰 때
다소 개인적이고 사소한 이슈였지만 IntelliJ를 주요 도구로 사용하는 입장에서 향후 이슈에 참고 할 만 하여 작성함  
JetBrain 계열의 도구들(PyCharm등) 이 공통적으로 참고할 수 있음

## IntelliJ의 Show Diff
Diff 기능은 아주 활용도가 높다. Clipboard나 history, 다른 파일과 비교할 때 정말 요긴하게 쓰인다.  
기본 동작은 Diff를 수행하면 새 dialog가 생성되며 두 내용을 비교하고, ESC를 눌러 끌 수 있다.  
이러한 행동이 익숙했던 내게 Diff화면이 main window의 새 탭으로 나오는 문제가 발생, 처음엔 일시적인 버그인가 하여 참고 쓰다가 도저히 불편해서(ESC로 닫히지 않음, 창이 아니니까) 원복을 시켜보고자 분투를 했었다.

## 문제
* 변경된 파일의 Show Diff를 누르면 Diff 화면이 새창(dialog)으로 안뜨고 tab으로 표시되는 문제
* 해결책을 찾고자 Google에서 수 시간 검색
  + 검색할 때 tab / window / dialog / popup / editor 등 정확한 용어를 알 수 없어 어려움
  + 공식 문서에도 해당 내용은 '당연한듯이' 없음

* 초기화 해볼까?
  + IntelliJ의 IDE Setting 초기화
  + File > Manage IDE Settings > Restore Default Settings... 를 하면 진짜 IDE가 최초 설치 상태로 돌아감
  + 즉, 설치된 plugins, 나에게 맞게 튜닝된 단축키/tab 구성, local history, Git Shelve(stash같은거) 등이 싹 다지워짐(이때 식은땀)
    + 다행히 기존 settings들을 .idea 폴더에 자동으로 백업해 줌(유료 도구 만세!)
    + 백업을 import하면 복구 가능
  + 어쨋든 초기화 했을 때, Show Diff는 원래대로 새창에 뜨게 원복됨
  + 프로그램 버그가 아닌 설정의 문제라고 확신

* 고객 지원에 연락
  + Show Diff와 관련된 모든 옵션을 다 수정해봤는데(도저히 관계 없어 보이는 것 까지) 시간만 잡아 먹음
  + Support에 ticket 등록(JetBrain 계정 필요)
    + 실수로 license가 없는 계정으로 신청했는데 빠르게 대응해 줌(주말 쉬고)
    + 보통 6시간 내에 답이 오는 듯 하고, 아마 license가 있는 계정이라면 좀 더 빠를 것 같음(24/7 ?)

## 고객 지원의 가이드
1. 최신버전으로 업데이트 해보라고 함
1. 로그와 스크린 캡쳐를 한 두 번 전달해서 확인 받음
1. Registry(Double shift > Registry)에서 설정을 변형해보라고 함 (로그에서 아래 값을 확인함)
    아래 값을 false로 변경, 비활성화 하면 해결
    ~~~
    show.diff.as.editor.tab=true
    ~~~~

## 정리
IntelliJ는 성숙한 유료도구답게 고객 지원이 잘되고, 설정값 들이 registry로 관리가 가능함  
만약 이와 유사한 문제를 만나면 IntelliJ 내부의 registry를 확인해 보자!


