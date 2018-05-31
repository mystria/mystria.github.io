---
layout: post
title:  "AWS ElasticBeanstalk에 Node.js app을 올려보자"
date:   2018-05-31 22:00:00 +0900
categories: AWS Node.js ElasticBeanstalk SSH
comments: true
---
# AWS ElasticBeanstalk에 Node.js app을 올려보자
ElasticBeanstalk(이하 EB)는 Web app을 배포하기 딱 좋게 사전 정의된 CloudFormation이다.  
EB가 지원하는 Web Application Server(WAS)는 여러가지가 있고, 주로 Tomcat을 써왔지만, 이번 과제에서는 불가피하게 유료 솔루션에서 제공하는 Node.js 서비스를 사용해야 하는 상황이 됐다.  
EB에 Node.js app을 배포하며 겪은 문제를 정리하고자 한다.

## 왜 하필 EB인가?

## EB 에서 Node command

## Express 모듈을 왜 안쓰고?

## PM2를 이용해야 하는 이유
멈추지 않는 EB Configuration update

## ebextensions를 이용한 후처리 추가

## node와 npm이 없네

## node version이 달라

## root 계정으로 보기
