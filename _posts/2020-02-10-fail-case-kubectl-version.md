---
layout: post
title:  "금주의 실패사례 - kubectl의 version문제"
date:   2020-02-10 18:00:00 +0900
categories: Kubernetes kubectl
comments: true
---
# Kubernetes의 자원(resource) 생성이 안되는 문제
Kubernetes 클러스터를 EKS를 이용해 구축하고, kubectl 설정까지 마친 후,  
node, pod, 등의 자원 조회는 잘 되는데 생성만 자꾸 안되어 삽질한 후기. 

## 발단
* Tutorial 급의 기본적인 기능은 잘 동작함
  + 클러스터에도 잘 연결됨
  + get, config 등 명령어도 잘 동작함
* 그런데 pod 생성만 안됨
  + 다른 사람이 만든건 잘 동작함
  ~~~ sh
    $ kubectl run busybox --image=busybox -it /bin/sh
    ...
    $ kubectl get pod
    NAME            READY     STATUS    RESTARTS   AGE
    busybox         0/1       Pending   0          50s
    others-pod      1/1       Running   15         6d
    $ kubectl describe pod busybox
    ...
    Events:
    Type     Reason            Age               From               Message
    ----     ------            ----              ----               -------
    Warning  FailedScheduling  50s (x3 over 2m)  default-scheduler  0/1 nodes are available: 1 Insufficient pods.
  ~~~
  + pending 상태에서 진행이 안됨
  + Network 문제나 EKS 설정 문제 등을 의심
    - image 가 없을지도? https://stackoverflow.com/questions/58257796/failedscheduling-0-3-nodes-are-available-3-insufficient-pods
    - ECR에 접속은 되나? https://docs.aws.amazon.com/ko_kr/AmazonECR/latest/userguide/ECR_on_EKS.html

## 경과
* kubernetes.io [공식 샘플](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)로 재시도
  + 색다른 에러 발생
  ~~~ sh
    $ kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml
    error: SchemaError(io.k8s.api.core.v1.NodeConfigSource): invalid object doesn\'t have additional properties

    $ kubectl apply -f shell-demo.yml
    error: SchemaError(io.k8s.api.networking.v1.NetworkPolicyPeer): invalid object doesn\'t have additional properties
  ~~~
  + 버전 문제라네? [Kubernetes create deployment unexpected SchemaError](https://stackoverflow.com/questions/55417410/kubernetes-create-deployment-unexpected-schemaerror)
* 버전을 확인해 보자
  + 생각해보니 옛날에 설치하고 한참 업데이트를 안함
  ~~~ sh
    $ kubectl version
    Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.5", GitCommit:"32ac1c9073b132b8ba18aa830f46b77dcceb0723", GitTreeState:"clean", BuildDate:"2018-06-22T05:40:13Z", GoVersion:"go1.9.7", Compiler:"gc", Platform:"darwin/amd64"}
    Server Version: version.Info{Major:"1", Minor:"14+", GitVersion:"v1.14.9-eks-c0eccc", GitCommit:"c0eccca51d7500bb03b2f163dd8d534ffeb2f7a2", GitTreeState:"clean", BuildDate:"2019-12-22T23:14:11Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}

    $ brew upgrade kubernetes-cli
    ...

    $ kubectl version
    Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.2", GitCommit:"59603c6e503c87169aea6106f57b9f242f64df89", GitTreeState:"clean", BuildDate:"2020-01-23T14:21:54Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"darwin/amd64"}
    Server Version: version.Info{Major:"1", Minor:"14+", GitVersion:"v1.14.9-eks-c0eccc", GitCommit:"c0eccca51d7500bb03b2f163dd8d534ffeb2f7a2", GitTreeState:"clean", BuildDate:"2019-12-22T23:14:11Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}

    $ kubectl apply -f shell-demo.yml
    pod/shell-demo created
  ~~~
* kubernetes-cli 버전을 올렸더니 해결

## 결론
서버(클러스터)와 클라이언타의 버전을 잘 확인하고, 적절한 버전으로 유지하자