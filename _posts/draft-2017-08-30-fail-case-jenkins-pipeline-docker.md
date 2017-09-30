---
layout: post
title:  "금주의 Jenkins 실패사례 - Pipeline"
date:   2017-08-30 12:00:00 +0900
categories: Jenkins Pipeline Failure
comments: true
---
# 이번 주 Jenkins를 사용하면서 겪은 실패 사례

## Jenkins build 파일 찾기
- 다른데서 build 수행한 것 copyArtifact로 가져오기
- 가져 온 것을 workspace로 복사
- 파일 이름을 변수로 읽어오기
- Jenkins의 환경 변수에서 알아낼 수 없을까? 확인 필요... Env.BuildUrl등

## pipeline과 node의 차이
- Declarative(선언적)와 Scripted
  - https://jenkins.io/doc/book/pipeline/syntax/
  - 예제를 보다보면 pipeline으로 시작을 많이 함 (2017년 2월에 나온듯)
    - https://jenkins.io/blog/2017/02/03/declarative-pipeline-ga/
    - https://stackoverflow.com/questions/42050626/jenkins-pipeline-agent-vs-node
    - 이게 더 좋은 건가?
  - 일단 기존에 쓰던 node로 작성
- Environment variables
  - pipeline으로 시작할 경우 environment 영역(?)으로 Env.variable을 정의할 수 있음
    - 대신 agent, stages, steps등 세세하게 작성해야 함
  - node로 시작할 경우 안됨
    - node 안에 직접 선언해야 함

## 변수의 사용
- stage간 변수 공유 안됨
  - node 단에 직접 선언해서 사용함 : 전역 변수?
  - sh '''의 경우 withEnv를 하지 않으면 전역 변수도 인식 안됨.. 이유를 모르겠음
  - pipeline 변수와 shell 상의 변수를 착각하지 말 것
    - 그런데 변수 사용할 때는 둘 다 $var 로 써야함
    - $var 와 ${var} 차이 확인 필요
    - https://stackoverflow.com/questions/6697753/difference-between-single-and-double-quotes-in-bash
- shell 실행 결과 변수로 반환
  - sh returnValue=true, script: ''' 사용시 반환되는 값은 0또는 1(성공/실패)
  - sh (returnStdout=true, script: ''').trim() 사용시 값 반환 됨 -> pipeline변수에 assign가능
    - https://issues.jenkins-ci.org/browse/JENKINS-26133
    - 기존에는 file에 쓰고 readFile함수로 값 가져왔는듯
      - https://stackoverflow.com/questions/37127815/how-to-set-the-output-of-sh-to-a-groovy-variable
- shell 에서 변수 선언
  - 변수명=값 : 띄어쓰기가 있으면 안됨, 변수명 규칙지켜야 함 -(hyphen)사용 불가
    - https://appcenter23.samsungcloudprint.com/
  - command결과 변수에 넣기
    - 변수명=$(command)
    - grep 사용 시 필요한 값 찾아 넣을 수 있음 (-d f 등 활용)
    - command 자체에서 filter 제공할 수 있음 (Go)
    - http://www.compciv.org/topics/bash/variables-and-substitution/
  - if / fi 문의 적절한 활용
    - 불필요한 sh command방지
  - 이 변수와 pipeline 내 변수와 착각하지 말 것

## Docker 사용시 이슈들
- 이런저런 문제                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
  - Docker를 Declarative pipeline에서 선언할 수도 있는 듯? 확인 필요
  - docker run
    - -it -rm 사용 불가
    - -network 활용 필요
- Docker container간 통신
  - link 사용 : deprecated 
    - https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/
  - network 사용 : 하나의 가상의 bridge로 연결하여 같은 네트워크 망에서 동작하는 것 처럼 사용 가능
    - https://docs.docker.com/engine/userguide/networking/work-with-networks/
- Server 기동 시간 기다리기
  - curl로 /healthcheck GET호출
  - waitUntil과 timeout 사용
    - https://stackoverflow.com/questions/37920830/make-jenkins-pipeline-wait-until-server-is-up
    - 이때 shell 실행 결과를 가져오는 걸 활용(curl로 OK값 반환 받는것 확인)
    
## Shell Scripts함
- sh
  - && 와 ;
  - |
  - \ : 행 마지막에 사용하면 command를 multiple line으로 쪼갤 수 있음(단 \ 뒤에 아무것도 없어야 함)

## Jenkins commands
- Pipeline steps reference
  - https://jenkins.io/doc/pipeline/steps/
    - https://jenkins.io/doc/pipeline/steps/workflow-basic-steps/
  - Jenkins Global variables
    - pipeline item > Pipeline Syntax > Global Variables Reference
