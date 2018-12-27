---
layout: post
title:  "금주의 실패사례 - Node.js 에서 package 설치 실패"
date:   2018-11-22 17:00:00 +0900
categories: Failure Apigee EdgeMicro Nodejs npm
comments: true
---

# Apigee Edge Microgateway 설치가 자꾸 실패해!
Apigee Edge는 API를 중계해주는 Cloud API 관리 플랫폼이다. SaaS로 사용 가능하지만 경우에 따라 별도로 설치하여 사용해야 하는 경우가 있는데, Apigee 전체 패키지를 설치하려면 엄청난 규모의 클러스터를 구축해야 한다.  
그래서 부가 기능(모니터링, 필터링 등)은 거의 제거하여 쉽게 설치/사용이 가능한 Apigee Edge Microgateway(이하 EdgeMicro)라는 Node.js기반의 설치형 서비스를 제공하고 있다.  
자세한 설명은 [공식 사이트](https://apigee.com/api-management/) 에서.. 참고로 Apigee Edge는 유료이다.  
이런 저런 사정으로 로컬머신에 Node.js와 EdgeMicro를 설치하려고 하고자 하였으나, 자꾸 설치가 실패하는 문제를 만났다.

## npm install 시 설치 경로
npm install <package> -g 로 설치된 package는 보통 (prefix인) /usr/local 아래의 /lib/node_module/<package>에 설치된다고 한다.  
https://docs.apigee.com/api-platform/microgateway/2.5.x/installing-edge-microgateway.html#whereisedgemicrogatewayinstalled  
그러나 실제로는 /usr/local이 아닌 /usr아래의 /lib/node_module/<package>에 설치되었는데,  
어떤 경우인지는 모르겠지만(nodejs나 npm의 버전 차이? global과 local의 차이?) NODE_PATH라는 ENV값(환경 변수로 변경 가능)의 default가 달라지는 것 같다.  
현재 prefix를 확인하고 싶으면 npm config get prefix 하면 path가 나온다.

## 사고 사례
  * 발단
    + 초기에 Node.js를 v10을 설치했다가 v6을 설치했다가 v8을 설치하는 등 버전을 오락가락함
    + edgemicro를 설치할 때 -g 옵션을 빼 먹었다가 다시 -g를 포함하여 설치함
    + 기타 등등 초보자짓을 많이함 (실제 Node.js 초보자임)
  * 문제 발생
    + edgemicro를 실행해보면 찾을 수 없다는 오류가 발생
      - 정확한 오류 메시지를 적어두지 않았네...  
      - bash: edgemicro: command not found (이런 느낌)
  * 문제 분석 
    + 왠지 모르겠지만, 실행 명령인 "edgemicro"를 수행해 보면 /usr/local/bin을 찾는다.
    + 여기에 link파일이 존재하고, 이 link는 /usr/local/lib/node_module/edgemicro 의 cli 실행파일(x 속성)을 가리킨다.
    + 즉, 우리가 shell에서 호출하는 edgemicro는 symbolic link인 것이다.
    + Node.js와 npm을 지우고 설치하기를 반복 보니 뭔가 꼬인 것 같은데, 
    + 새로 설치한 환경에서는 /usr/bin에 link파일이 존재하고, 이 link는 /usr/lib/node_module/edgemicro 의 cli 실행파일을 가리킨다.  
    + 즉, prefix(기존에는 /usr/local, 현재는 /usr)에 따라, 설치 및 실행파일 위치가 달라진다는 점이다.
  * 문제 원인 
    + 다음과 같이 실행 명령의 type(위치)을 확인하면 /usr/bin이 찾아진다.
      > $ type -a edgemicro 
    + 그러나 불행히도 실행 명령은 /usr/bin보다 /usr/local/bin을 더 먼저 찾기 때문에, /usr/local에 설치 된 것이 있다면, /usr에 설치된 것은 없는 것 처럼 인식된다.
      - 혹시 이것은 환경변수 PATH에 등록된 순서때문인가? (/usr/local/bin이 /usr/bin보다 앞에 있었음)  

## 해결 방법
가장 좋은 방법은 npm uninstall 보다 직접 두 군데의 파일을 완전히 삭제하는 것이다.  
  * Node.js 깔끔하게 지우는 방법
    + https://www.theteams.kr/teams/35/post/67342
  * 왜 설치경로가 다를까?
    + [/bin, /usr/bin, /usr/local/bin 의 차이점](http://wookiist.tistory.com/10)
    + 기본적으로 용도 및 실행 scope(?)의 차이
    + 정확한 정리는 아니지만 대략 느낌으로 나타내자면,
      - /bin : 시스템 프로그램
      - /usr/bin : 사용자용 프로그램
      - /usr/local/bin : 현재 사용자용 프로그램
      - 정확한 정보는 [공식? reference](http://www.pathname.com/fhs/) 참조

## 기타 주의사항
그리고 edgemicro는 latest버전이 동작 하지 않을 수 있으므로 안정화된 버전을 찾아서 설치 할 것!
즉, npm install edgemicro@2.5.7 과 같이 버전 명시를 생활화 해야한다.

## 참고
  * Ubuntu계열에서 Node.js 설치 방법
    + https://github.com/nodesource/distributions/blob/master/README.md
  * 명령어의 위치 찾는 방법
    + https://stackoverflow.com/questions/2869100/shell-how-to-find-directory-of-some-command
  * PATH에 설치 경로를 추가하거나, prefix를 변경하여 해결하는 방법
    + https://stackoverflow.com/questions/15054388/global-node-modules-not-installing-correctly-command-not-found
  * Node.js의 global 설치 관련 글
    + https://github.com/nodeschool/discussions/wiki/Installing-global-node-modules-(Linux-and-Mac)
    + 위에 언급한 해결책에 대해서도 정리되어 있음
  * 조금 오래됐지만, global 설치와 local 설치의 차이점 설명
    + https://nodejs.org/en/blog/npm/npm-1-0-global-vs-local-installation/
    + [번역된 글](http://blog.doortts.com/226)
  * NODE_PATH 와 관련된 문의들
    + https://stackoverflow.com/questions/26293049/specify-path-to-node-modules-in-package-json
    + https://stackoverflow.com/questions/7076529/how-to-set-node-path-for-nodejs-ubuntu
  
## 맺음말
이번 글은 정리가 잘 안되었습니다. 그냥 비슷한 문제를 다시 만나게 되면 참고하기 위해 주저리 모아놨습니다.

