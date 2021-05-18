---
layout: post
title:  "git push -f 로 사라진 commit 복구"
date:   2021-03-30 24:00:00 +0900
categories: git
comments: true
---

# 공포의 git push -f
살다보면 force push (git push -f) 를 쓸 일이 있다. 그럴 일이 없는게 제일 좋겠지만, 뭔가 잘못 commit 하고 push 한 걸 흔적없이 깔끔하게 되돌리고 싶은 욕구, 꼬여버린 git history graph 를 좀 보기좋게 돌려놓고 싶은 욕구가 생기곤 한다.  
하지만 force push 는 너무 위험하기 때문에 모두가 말린다. 남들이 말리는건 듣는게 좋겠지만, '나는 괜찮겠지' 하는 몹쓸 합리화로 누군가는 쓰고야만다. 그리고 실수를 하게 되고, 무언가를 날려먹은 이후에 이하 본문에서 서술할 force push 를 되돌릴 방법을 검색하게 되곤 한다.  

## git push -f 로 망쳐버린 commit을 살려보자
* 왜 이런일이 생길까?
  - git reset --hard 로 과거로 돌아감
    - 여기서 --hard 말고, mixed나 soft로만 돌려도 다행이었을텐데 ...
  - git push -f 로 과거 시점으로 commit을 돌림 
    - 이때 모든 history가 사라지고 commit간의 link도 dangling 된다.
    - 즉, history가 없기 때문에 간단하게 push를 되돌릴 수 없다. log를 보면 그저 현재 commit으로 부터 과거만 볼 수 있다.
* 확인해 보자!
  - 모든 것이 사라지고 현재 HEAD는 c076328d6 을 가리키고 있다. (본문의 sha-1 hash는 예시이며, 모두 다름)
    ``` bash
      your-project$ git log -5 --pretty=format:"%h - %an, %ar : %s"
      c076328d6 - yourname, 77 minutes ago : commit 5
      74f9f15e1 - yourname, 11 hours ago : commit 4
      b9e9ac012 - yourname, 3 days ago : commit 3
      a1c6ed523 - yourname, 3 days ago : commit 2
      977fe4914 - yourname, 3 days ago : commit 1
    ```
  - 정상처럼 보인다. 아주 깔끔하게 사라졌기 때문에 문제를 못느낄 수도 있다. 다만 코드가 옛날로 돌아가 있을 뿐... c076328d6 이후의 commit들은 사라졌다. 되돌리고 싶어도 되돌아 갈 곳이 없어진 것이다.

* 응급 조치
  - **절대 git gc(garbage collection)를 하면 안된다**. gc를 하면 dangling 된 commit이 삭제될 수 있다.
  - IDE를 사용한다면, 해당 IDE의 local history를 이용해서 긴급 복구도 가능하다. IntelliJ 같은 도구에는 workspace의 작업 이력을 어느 정도 저장해주곤 한다. 모든 IDE가 그렇지는 않으므로 일단 확인해보자. (가급적 좋은 IDE를 쓰자)
  - 복구 작업 전에 이 local history로 복구한 내용을 (소스코드만) 백업해두면, 최악의 경우 history는 잃어도 작업물은 지킬 수 있다. (commit을 squash 한거라고 정신 승리 가능)

* 이제 다른 사람의 실수를 보고 배우자
  - 아래 링크에서 같은 길을 걸었던 사람들의 해결책을 확인할 수 있다.
    - [The illustrated guide to recovering lost commits with Git](http://www.programblings.com/2008/06/07/the-illustrated-guide-to-recovering-lost-commits-with-git/)
    - [Stack Overflow: How can I recover from an erronous git push -f origin master?](https://stackoverflow.com/questions/3973994/how-can-i-recover-from-an-erronous-git-push-f-origin-master)
  - 의외로 사람들이 해결책이 간단하다고 한다. git을 개발한 토발즈 님께 감사드립니다.

## 해결책
* 작업 이력 확인
  - 먼저 지난 git terminal log(git 수행이력)를 확인해 보자
  - 
  <!-- -->
    ``` bash
      your-project$ git push -f
      Total 0 (delta 0), reused 0 (delta 0)
      To https://github.com/your-org/your-project.git
      + cca7fffb4...c076328d6 MY_BRANCH -> MY_BRANCH (forced update)
    ```
  - 내가 한 멍청한 짓을 로그로 확인 가능하다. 
  - 다행히 망치기 직전의 commit hash 값을 확인 할 수 있다. (여기서는 cca7fffb4) 이 값을 아는게 중요한데, 이 값이 우리가 돌아가야할 목적지이기 때문이다.
  - 참고로 github web에서 이 hash를 통해 코드와 히스토리가 아직 남아 있음을 확인할 수 있다.
    - https://github.com/your-id/your-project/commits/**cca7fffb4**
  - 만약 이 log를 확인할 수 없다면, 별도의 방법으로 마지막 commit hash를 찾아야 한다.
    - git reflog, git fsck 명령 등으로 확인 가능
    - [Git의 내부 - 운영 및 데이터 복구](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EB%82%B4%EB%B6%80-%EC%9A%B4%EC%98%81-%EB%B0%8F-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B3%B5%EA%B5%AC)
  - 
    ``` bash
      your-project$ git reflog show remotes/origin/MY_BRANCH
      c076328d6 (HEAD -> MY_BRANCH, origin/MY_BRANCH) remotes/origin/MY_BRANCH@{0}: update by push
      cca7fffb4 remotes/origin/MY_BRANCH@{1}: update by push
      2267b0ddf remotes/origin/MY_BRANCH@{2}: update by push
      b32cdbd13 remotes/origin/MY_BRANCH@{3}: update by push
      f8f05d02f remotes/origin/MY_BRANCH@{4}: update by push
      1b440e922 remotes/origin/MY_BRANCH@{5}: update by push
      0aa4b7df4 remotes/origin/MY_BRANCH@{6}: update by push
    ```
* 이제부터 집중해서 따라하기
  - 우리는 commit hash 주소로 checkout이 가능하기 때문에, 직전 commit으로 돌아갈 수 있다.
  - 위에 [Stack Overflow의 두번째 답변](https://stackoverflow.com/a/24373376/8350542)이 정답
    ``` bash
      your-project$ git checkout cca7fffb4
      Note: checking out 'cca7fffb4'.

      You are in 'detached HEAD' state. You can look around, make experimental
      changes and commit them, and you can discard any commits you make in this
      state without impacting any branches by performing another checkout.

      If you want to create a new branch to retain commits you create, you may
      do so (now or later) by using -b with the checkout command again. Example:

        git checkout -b <new-branch-name>

      HEAD is now at cca7fffb4... Merge remote-tracking branch 'origin/master' into MY_BRANCH
    ```
  - 이제 이 checkout을 다시 원래의 branch로 붙이면 된다. 안붙여진다면 -f 로 강제로 붙여주자. (또 force?)
    ``` bash
      your-project$ git branch MY_BRANCH
      fatal: A branch named 'MY_BRANCH' already exists.

      your-project$ git branch MY_BRANCH -f
    ```
  - 지정한 branch에 잃어버렸던 commit이 연결된다. 이제 그 branch로 다시 checkout 하고, origin으로 push 하자.
  (이 부분은 각자 조금씩 다를 수 있으니 상황에 맞춰서 진행 필요)
    ``` bash
      your-project$ git checkout MY_BRANCH
      your-project$ git push origin refs/heads/MY_BRANCH:MY_BRANCH
    ```

## 결론
요약하자면, push -f 로 dangling 되어버린 commit을 다시 내 branch로 이어주자는 것이다.
force push 는 안쓰는게 좋겠지만, 어쨌든 쓰라고 있는 기능이니 조심해서 쓰면 된다. 이제 되돌리는 방법도 알았으니(!) 너무 겁먹지 말자.
