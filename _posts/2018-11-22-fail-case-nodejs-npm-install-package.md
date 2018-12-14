---
layout: post
title:  "NodeJs"
date:   2018-11-22 17:00:00 +0900
categories: Kubernetes EKS EIP AWS
comments: true
---
# EKS로 동작하는 K8s Pod에 EIP를 붙여 보자
Kubernetes에서는 Service를 통해 cluster 내에서 동작하는 서비스를 외부로 노출한다.  
(서비스를 Service로 노출하다니!)  
이 Service는 platform의 영향을 받게 되는데, EKS는 Load Balance로 할 경우 Elastic Load Balancer를 붙여 준다.  
그 외에도 Cluster IP나 Node Port 등이 존재하는데, 이에 대한 설명은 생략하고, Elastic IP를 붙일 순 없을까 생각해 봤다.  

## Pod에 고정 IP 필요한 경우

## Pod 정보 받기

## Pod의 Private IP를 이용해 ENI찾기
ENI(Elastic Network Interface)

## EIP 설정
설정할 때 Instance가 아니라 Network로 해서 ENI에 연결


no basic auth credentials일때 aws-auth-cm.yaml 다시 실행

npm install <package> -g 로 설치된 package는 보통 (prefix인) /usr/local 아래의 /lib/node_module/<package>에 설치된다고 한다.  
https://docs.apigee.com/api-platform/microgateway/2.5.x/installing-edge-microgateway.html#whereisedgemicrogatewayinstalled
그러나 실제로는 /usr아래에 /lib/node_module/<package>에 설치되었는데, 
어떤 경우인지는 모르겠지만(nodejs나 npm의 버전 차이? global과 local의 차이?) NODE_PATH라는 ENV값(환경 변수로 변경 가능)의 default가 달라지는 것 같다.  
현재 prefix를 확인하고 싶으면 npm config get prefix 하면 path가 나온다.

- 사고사례
왠지 모르겠지만, edgemicro를 수행해 보면 /usr/local/bin을 찾는다. 
여기에 link파일이 존재하고, 이 link는 /usr/local/lib/node_module/edgemicro 의 cli 실행파일을 가리킨다.  
nodejs와 npm을 지우고 설치하다보니 꼬인 것 같은데, 새로 설치한 환경에서는  
/usr/bin에 link파일이 존재하고, 이 link는 /usr/lib/node_module/edgemicro 의 cli 실행파일을 가리킨다.  
즉, prefix에 따라, 설치 및 실행파일 위치가 달라진다는 점이다.
그러나 불행히도 실행파일을 /usr/bin보다 /usr/local/bin을 더 먼저 찾기 때문에, /usr/local에 설치 된 것이 있으면, global로 설치된 것은 없는 것 처럼 인식된다.
반면, type -a edgemicro 와 같이 실행 파일의 type(위치)을 확인하면 /usr/bin가 찾아지기 때문에 있는데 왜 안되는지 알 수 없는 혼란을 겪게된다.
혹시 이것은 환경변수 PATH에 등록된 순서때문인가? (/usr/local/bin이 /usr/bin보다 앞에 있었음)

가장 좋은 방법은 npm uninstall 보다 직접 두 군데의 파일을 완전히 삭제하는 것이다.
그리고 edgemicro는 latest버전이 동작 하지 않을 수 있으므로 안정화된 버전을 찾아서 설치 할 것!
즉, npm install edgemicro@2.5.7 과 같이 버전 명시를 생활화 해야한다.


참고: Ubuntu계열에서 Node.Js 설치
https://github.com/nodesource/distributions/blob/master/README.md

https://stackoverflow.com/questions/2869100/shell-how-to-find-directory-of-some-command
Prefix를 수정하거나 실행 경로를 shell에 정의:
https://stackoverflow.com/questions/15054388/global-node-modules-not-installing-correctly-command-not-found

참조: Node.Js의 Global 설치 관련 글
https://github.com/nodeschool/discussions/wiki/Installing-global-node-modules-(Linux-and-Mac)
NODE_PATH 변경
https://stackoverflow.com/questions/26293049/specify-path-to-node-modules-in-package-json
https://stackoverflow.com/questions/7076529/how-to-set-node-path-for-nodejs-ubuntu