---
layout: post
title:  "두 Repository 합치기 - 분리 후 재결합"
date:   2020-06-01 24:00:00 +0900
categories: Git
comments: true
---

# 두 repository 합치기(merge)
Repository(이하 repo)를 합치는 방법은 인터넷에 검색해보면 많이 찾을 수 있다.  
그러나 보통 한 repo에서 분리되어 나왔다가 다시 합치려는 경우(1)를 설명하고 있기 때문에 개인적으로 경우(2)에는 어떻게 해야 하는지 궁금했다.
* 경우(1)
~~~
Repo 1 : Rev1 <- Rev2 <- Rev3 <- Rev4 <- ...
Repo 2 :               \ Rev3 <- Rev4 <- ...
을
Repo 1 : Rev1 <- Rev2 <- Rev3 <- Rev4 <- Rev7 <- ...
                       \ Rev5 <- Rev6 /
으로 합치기
~~~
* 경우(2)
~~~
Repo 1 : Rev1 <- Rev2 <- Rev3 <- Rev4 <- ...
Repo 2 :                 Rev1 <- Rev2 <- ...
와 같이 history 다 날리고 새로 만든 Repo 2 를 합치기
~~~
어쨋든 파일 이름만 동일하지 서로 아무 관계 없는 repo를 합치려는 시도는 어떻게 될 것인가?

## 결과
결과적으로 합쳐지긴 하지만 뿌리가 갈라진 모양으로 합쳐지기 때문에 이력을 추적하기 힘들다(불가능 하진 않다).
~~~
Repo 1 : Rev1 <- Rev2 <- Rev3 <- Rev4 <- Rev7 <- ...
                         Rev1 <- Rev2 /
~~~

## 작업 방법
어디에 있지...
