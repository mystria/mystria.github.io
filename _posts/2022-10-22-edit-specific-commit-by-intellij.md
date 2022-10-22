---
layout: post
title:  "IntelliJ에서 과거 특정 커밋 수정하기"
date:   2022-10-22 20:00:00 +0900
categories: IntelliJ Git
comments: true
---


# IntelliJ에서 과거 특정 commit 수정하기

Git 의 log를 예쁘게 관리하면 훗날 변경 이력을 검토할 때 도움이 된다. 언제 누가 왜 이런 변경을 했는지 추적을 해야 할 일(없으면 좋겠지만)은 계속 발생하기 때문이다. 

물론 log를 예쁘게 관리하는데 연연하지 않고 빠르게 수정하며 앞으로 나아가는 것도 좋은 방법이다.  
하지만 실수로 빠트린 코드 한 줄 때문에 이전 commit들의 build가 깨져 불편하다거나 불필요한 commit log가 한 줄 생기는게 영 껄끄럽다면 과거로 돌아가 특정 commit만을 수정해볼만도 하다.

엄밀히 말하자면 과거로 돌아가 특정 부분만 살짝 수정하는 것이 아니라 과거로부터 새롭게 역사를 쓰는 것이다. 그렇다면 기존의 역사(log)는 어떻게 될까? 안타깝게도 기존 log는 사라져야 한다. 즉, force push를 하여 기존 commit의 hash가 모두 새로 만들어지는 것이다.

마치 시간 여행으로 과거를 바꾸면 멀티버스가 생성되지만 절대자에 의해 다른 우주는 소멸되는 것에 비유할 수 있다. 그러므로 기존 우주(?)에서 작업 중인 동료가 있다면 새로운 우주로 바뀔 것을 미리 알려주어야 할 것이다.

# IntelliJ에서 제공하는 git 제어

IntelliJ에서 다양한 git 기능을 제공한다. git 명령어로는 다소 복잡한데다 실수 할 수도 있기 때문에 IntelliJ의 UI로 작업을 해보자. (IntelliJ IDEA 2021.3.2, Ultimate Edition 기준)

## Amend

> Sidebar → Commit → Commit to branch → Changes
> 

만약 수정 대상이 직전 commit이라면 `amend` 를 통해 변경 사항을 덧붙여 commit을 다시 만들 수 있다. 물론 이 또한 기존 commit은 삭제되어야하므로 force push가 필요하다. 직전 commit만 변경되므로 아주 간단하고 파급력도 적다고 볼 수 있다.

방법은 간단하다. IntelliJ의 Commit 탭에서 Amend를 체크하면 된다. 체크하는 순간 기존 commit의 commit message가 나타나기 때문에 ‘아 기존 commit이랑 합쳐지겠구나’하는 느낌이 바로 올 것이다.  
Amend를 체크하고 commit한 다음, push 할 때 Force Push 버튼(Push 버튼에서 변경)을 누른다.

화면을 캡쳐하면 좋겠지만, 일단 이렇게 구두로만 설명해둔다. (* 주의: IntelliJ의 버전에 때라 Amend 체크박스의 위치가 다를 수 있다.)

## Reset Current Branch to Here…

> Tool bar → Git → Log → 대상 commit을 우클릭 → Reset Current Branch to Here…
> 

무식하지만 간단한 방법도 존재한다. 수정하려는 commit 이전으로 `reset` 하고, 새롭게 commit을 쌓아가는 방법이다. reset할 때 soft나 mixed 모드를 선택하면 변경된 코드가 커밋하지 않은 상태로 남아있기 때문에, 원하는 대로 수정하고 새로운 commit log를 만드는 것이다.

이 방법은 commit 2~3개 정도까지는 큰 문제가 없지만 오래된 과거로 거슬러 올라갈 수록 수정 파일이 겹치거나 복잡해지므로 사실상 불가능하다. 게다가 새로 commit을 하게되므로 commit한 시간과 작성자 이력이 내가 방금 한 것처럼 변경된다. (물론 git 명령의 옵션을 통해 이력을 커스텀 할 수 있지만 손이 많이간다. IntelliJ UI 에서는 작성자만 변경가능하다.)

그냥 이런 방법도 있다… 정도로만 설명하고 넘어간다.

**과거 특정 commit만 수정하는 것이 목표이기 때문에 다른 commit 들은 그대로 유지되도록 하자!**

## Interactively Rebase from Here…

> Tool bar → Git → Log → 대상 commit을 우클릭 → Interactively Rebase from Here…
> 

과거 commit을 수정하는 것은 다르게 말하면 `rebase` 를 새로 하는 것이다. 예를들어 수정하려는 commit을 A라고 한다면, A 이전 커밋으로 돌아가 새로운 A(=A’)를 만들고 A’ 위에다 이후 commit 들을 rebase 하는 것이다.

즉, 아래와 같이 이어지는 log가 있다면, 

> 1 → 2 → A → 3 → 4
> 

2로 돌아가 A를 수정(amend)하여 새로운 A’ commit을 만들고 이후 3, 4를 rebase하여, 

> 1 → 2 → A’ → 3’ → 4’
> 

이렇게 새로 만드는 것이다. 이때 기존 A → 3 → 4는 force push로 인해 사라지게 된다.

### Interactively Rebase

Interactively Rebase from Here… 을 수행하면 수정을 위한 Rebasing Commits 팝업창이 표시된다. 선택한 commit에서 최신 commit까지 위에서 아래로 목록으로 표시되는데, 여기서 다양한 작업을 할 수 있다.

- 🔼/🔽 (Up/Down): commit의 순서를 변경 (서로 상관없는 commit이면 문제 없으나 변경이 겹칠 경우 conflict 가 발생할 수 있으므로 주의하자)
- ↩️⮌ (Pick): 해당 커밋은 아무것도 하지 않고 그대로 유지, 만약 뭔가 수정했었다면 이를 되돌림
- ⏸ (Stop to Edit): 이 commit에서 rebase를 멈춤 → **이 때 우리가 commit의 코드를 수정할 수 있다.**
- Reword: 선택한 commit의 commit message 수정
- Squash(or Fixup): 선택한 commit을 이전 commit에 통합 (Fixup은 기존 commit message 사용)
- Drop: 선택한 commit을 삭제(즉, 변경 내역이 삭제됨)
- Reset: 이 팝업에서 작업한 내용 모두 초기화

이 기능들은 다음 블로그에서 git 명령어로 설명되어 있으니 참고하자.

- [https://wormwlrm.github.io/2020/09/03/Git-rebase-with-interactive-option.html](https://wormwlrm.github.io/2020/09/03/Git-rebase-with-interactive-option.html)

### 특정 commit 수정

수정하려는 commit(이하 A)을 선택하여 일시정지를 누르고, (혹시 또 다른 작업이 필요하면 설정하고) Start Rebasing 버튼을 누른다. 이제 단계별로 rebase가 진행될 텐데, 우리가 선택한 commit A에서 일시정지된다.

그러면 A에 checkout 된 상태가 되고 아마 우측하단에 rebase를 계속(continue)하거나 취소(abort)할 수 있는 팝업이 표시된다.

앞서 이야기한 `amend` 를 이용하여 A에 수정 사항을 반영한 다음, 우측하단의 continue를 눌러 나머지 commit들을 마저 rebase 한다. 이때 수정한 내용이나 작업에 따라 conflict가 발생할 수 있으므로 적절히 merge하여 해결한다. 모든 rebase가 완료되면 작업은 끝난다.

이제 이 commit들을 origin으로 push하는 일이 남았다.

### Force Push

앞에서 설명했지만 현재 작업한 내용은 local에만 적용된 상태이며 origin에는 변경 이전 내용이 남아있다. 동료 개발자가 있다면 여전히 그 origin을 참조하여(정확히는 origin의 clone) 작업을 진행 중일 것이므로 내가 작업한 내용을 force push하면 동료의 작업이 사라질 수 있기 때문에 조심하자.

다르게 말하자면 force push 하기 전까진 기존 log가 origin에 남아 있을 것이므로, 안심하고 다양하게 시도해보자. 제대로 안되면 그냥 rollback하면 되니깐!

Push 옵션으로 `--force` 보다 `--force-with-lease` 을 주면 조금 더 안전한데, 다행스럽게도 IntelliJ의 UI에서 Force Push는 force-with-lease 이다. with lease는 push 하려는 branch에 다른 사람의 commit 이 이미 있을 경우 force push를 멈춰주는 기능이다. IntelliJ 에서는 다음과 같이 force push가 rejected 되었다고 표시한다. 

> Push rejected, Force-with-lease push {branch} to origin/{branch} was rejected
> 

### 주의 사항

- 만약 force push를 하지 않고 그냥 push를 한다면 괜히 새로운 branch만 한 줄 더 생긴 꼴이되니 지금까지 작업은 아무 의미가 없어지게 된다.
- 만약 수정(rebase)할 commit 이 후에 갈라져 나간 다른 branch가 있다면, 그 branch에서 부터 거슬러올라가는 기존 branch가 사라지지 않고 남아있을 것이다. 해당 branch도 내가 새로 force push로 만든 branch 위에 rebase 해줘야 한다.

# 결론

과거의 특정 commit 을 수정하는 작업이라 설명했지만, 사실 기존 commit 들을 동일한 정보로 다시 commit 하는 것이다. 동일한 commit 정보로 새로운 commit 을 만든 것이므로 기존 commit 들을 없애야 하는 문제도 있다.

그러나 이 기능을 알고 난 이후에는 git 작업을 하는데 자신감이 많이 붙었다. 이전에는 잘못 push 하거나 누락한 것이 있으면 git log가 엉망이 되기 때문에 commit 하나 하나에 신경이 곤두섰다. 하지만 이제는 (동료와 합의만 된다면) commit 의 message를 수정하거나 순서를 바꾸거나 누락된 것을 추가할 수 있게 됐다. 이 기능을 쓰지 않고 알고만 있더라도 든든할 것이다.

작업 절차를 간단히 정리해 둔다. Terminal에서 git 명령어로 작업하는 절차도 이와 같다.

1. git log 확인
2. interactively rebase 로 대상 commit 지정 (대상 commit 직전으로 head 이동)
3. 지정된 commit을 amend
4. 이후 나머지 commit 들을 rebase
5. force push 로 기존 commit 지우기 (commit 간 연결 해제)

# Git 명령어로 작업하기

어떤 사람들은 terminal을 통해 git을 이용하기도 한다. 명령어로 작업하면 훨씬 강력하지만 익숙치 않은 사람에겐 terminal에서 text로만 표시되는 수많은 정보가 직관적으로 이해되지 않아 실수를 유발할 수 있다.

그렇지만 git 명령이 ‘근본’이기 때문에 알아 두는것도 좋을 것이다. 아래 블로그를 읽어 보는 것을 추천한다.

- [https://homoefficio.github.io/2017/04/16/Git-과거의-특정-커밋-수정하기/](https://homoefficio.github.io/2017/04/16/Git-%EA%B3%BC%EA%B1%B0%EC%9D%98-%ED%8A%B9%EC%A0%95-%EC%BB%A4%EB%B0%8B-%EC%88%98%EC%A0%95%ED%95%98%EA%B8%B0/)
- [https://hbase.tistory.com/145](https://hbase.tistory.com/145)