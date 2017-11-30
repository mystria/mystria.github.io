---
layout: post
title:  "서버 모니터링 도구 정리"
date:   2017-11-29 21:00:00 +0900
categories: Monitoring DevOps
comments: true
---
# 모니터링 도구 정리
서버 모니터링 도구 몇가지를 찾아서 정리해 봄.  
기준을 갖고 체계적으로 정리하고 싶었지만, 짧은 영어 실력때문에 수집되는대로 막정리.  
틀린 정보도 있을 수 있으니 참고만 할 것. \*표시는 사견임.  
(틀린 사항이 있으시면 댓글 부탁드립니다.)

## Nagios
  * 개요 : http://sepiros.tistory.com/17
    + Official : https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/about.html#whatis
    + 오픈소스 서버 / 모니터링 프로그램
    + Nagios Server + Nigios Agent/Target
    + 오픈소스: Nagios Core / 상용: Nagios XI
  * 동작에 관한 설명 : http://system-monitoring.readthedocs.io/en/latest/nagios.html
  * Addons : https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/addons.html
  * 호스트->원격시스템 질의, 원격시스템->호스트 보고, 조건에 따라 Noti
  * 원격시스템 질의 : NRPE (Nagios Remote Plugin Executor) : 5666 port
    + SSH로 shell command를 보내고 exit코드를 받는 것과 같은 원리
    + 원격 장치의 Plugin을 실행 시켜 결과를 반환 받아 체크
      - Plugin은 Nagios Exchange에서 받을 수 있음
    + 원격 장치에 위치(Executor이므로)* : apt-get install nrpe
  * 호스트 보고(Passive Check) : NSCA(Nagios Service Check Acceptor) : 5667 port
    + 원격 장치와 app에서 Passive alert과 check를 통합할 수 있게 해주는 daemon
    + 모니터링 호스트에 위치하는 듯(Acceptor이므로)*
      1. "External Application"이 서버나 서비스의 상태를 체크
      2. "External Application"이 결과를 "External Command File"에 기록(외부에서 timestamp로 작성, 권한설정 필요)
      3. Nagios(External Command Logic)가 external command file을 읽어 모든 passive check info를 Queue(Active check, Passive check 공용)에다 넣음
      4. Nagios가 주기적으로 reaper event(?)를 실행하고, queue를 스캔함. Queue에 check 결과가 있으면 (passive건 active건 상관없이) 처리
      5. check 결과에 대한 action(Notifications 등) 수행
      * Active check와 Passive check의 처리 방식은 동일(= Seamless integration)
  * Nagios의 장단점 : http://server-talk.tistory.com/120

## Munin
  * 북유럽신화 속 오딘의 까마귀 전령(Hugin & Munin)중 하나, 기억을 의미 : 이름 참 잘지음.
  * 개요 : http://munin-monitoring.org/wiki(Official), http://guide.munin-monitoring.org/(Guide)
    + Perl based
    + RRD도구를 사용, Master-Node 아키텍쳐, Master -> Node 질의 / RDD저장 / Graph화
  * Protocols
    + Poller-based
    + Asynchronous proxy node : 옛날 Munin은 Synchronous였는데, 성능을 위해 병렬화, 비동기화 시킴 -> proxy가 필요
    + SNMP Plugins : Simple Network Management Protocol : 161 port : 네트워크 관리를 위한 표준 프로토콜
    + Common Network : RMT(Munin Resource Monitoring Tool) 를 이용해 command를 Node로 전달, Node의 daemon이 이해하여 응답 : 4949 port
    + 모니터링 성능 향상을 위한 네트워킹 기술들
      - Multigraph(v1.4) : 동시에 여러개
      - Dirtyconfig(v2.0) : 한번에 설정과 값을 동시에
    + Nagios와 꿀조합
      - NSCA(Nagios Service Check Acceptor)를 통해 Nagios와 통신 : 5667 port : Nagios 참고
      - Munin에서 munin-limits를 통해 Nagios로 NSCA를 통해 메세지(OK/Warning/Critical/Unknown) 전달
  * Master
    + munin-cron : Cron, munin-limits와 munin-update를 실행
    + munin-update : Data 수집기, Munin node로 부터 데이터를 가져와 RRD에 저장
    + munin-limits : 상황 발생시 알림, 주로 Nagios의 연락처와 연동
  * Node : Master에도 스스로를 모니터링하기 위해 Node 필요
    + Node가 없어도 기본적인 것은 모니터링 가능
  * Plugin
    + 원격 서버에서 정보를 얻기위해 실행되며, host는 SNMP를 이용해 결과를 가져감
    + config를 통해 metadata와 value를 구분해서 얻을 수 있음(두번 통신?)
    + Apache 웹서버의 모니터링지원 Plugin 존재
  * 설치 가이드
    + https://www.digitalocean.com/community/tutorials/how-to-install-the-munin-monitoring-tool-on-ubuntu-14-04
    + http://apollo89.com/wordpress/?p=3964
  * 개인적인 평가 : Nagios가 전체를 한 눈에 볼 수 있다면, Munin은 개별을 세세하게 볼 수 있음

## Cacti
  * 개요 : http://server-talk.tistory.com/120
    + Official : https://www.cacti.net/what_is_cacti.php
    + 데이터를 지닌 Script나 Command의 경로를 Cacti로 전달하면, Cron job으로 MySQL이나 RRD에 저장
    + SNMP로 통신 : 161 port
    + 다양한 Plugin지원
  * PHP based
  * RRD도구의 FrontEnd
    + RRD도구 : Round-Robin Database Tool : MRTG로부터 유래
      - MRTG : Multi Router Traffic Grapher : 인터넷회선의 사용량을 SNMP를 통해 수집, 그래프화
      - RRD도구는 트래픽이 아닌 시스템(서버)의 일반적인 정보(온도, 속도, 전압 등)를 수집, 그래프화
      - Circular Queue(Buffer)에 데이터를 읽고 씀
  * 개인적인 평가 : 가장 단순한 서버 모니터링

## Chef
  * 개요 : https://docs.chef.io/chef_overview.html
    + 우리말 : http://devopser.me/chef-devops-yi-ggoc/, https://www.ibm.com/developerworks/community/blogs/9e635b49-09e9-4c23-8999-a4d461aeace2/entry/215?lang=en, 많이 쓰이는 도구로서 자료가 많음
    + Ruby based
    + Chef Workstation(Chef Client) -> Chef Server -> Chef Node(Chef Client)
    + Recipe : 서버를 제어할 로직
      - Databag : Recipe가 참조할 데이터
    + Cookbook : Recipes + 기타 정보(Resource, File 등), 배포의 기본단위
      - Role : Cookbook속 Recipe의 조합
    + Knife : CLI
    + (모니터링이 가능한) 원격 서버 관리 프레임워크
      - Recipe로 이루어진 Cookbook을 Knife로 Chef Client에 수행시켜 서버의 설정 및 관리를 자동화
      - 원격으로 SSH를 병렬로 수행하는 것과 유사
      - 시스템 정보 획득 가능(모니터링) : 주 목적이 모니터링이 아님
  * 설정
    + 관리할 서버에 Client 설치
    + Chef Server에서 Client 등록, Server와 Client간 통신할 SSL인증서 등록
    + Client로는 서버 정보 읽기만 가능, 업데이트 하려면 권한 필요(Development Kit을 통해 ACL관리 가능)
  * AWS에서 활용
    + EC2에서 Chef Client가 설치된 AMI를 기반으로, User-Data(Cloud-init)를 이용하여 세부설정(hostname, role등)
    + Chef Server를 통해 AWS내 AMI들을 관리(버전 업데이트, 보안 설정 등)
  * 개인적인 평가 : 완성도 높은(사용자 많은) 서버 관리 도구

## MongoDB Monitoring System(MMS)
  * 현재는 MongoDB Cloud에서 서비스 중 : MongoDB Cloud Manager(https://docs.cloudmanager.mongodb.com/)
    + 유료
  * 기존 설치형 MongoDB 모니터링은 : http://api.mongodb.com/mms/current/
    + Agent가 설치되어 mongod와 mongos를 모니터링, seed 정보를 이용하여 다른 node의 MongoDB도 모니터링 가능
      - Agent가 MMS로 데이터를 SSL을 이용해 전달(HTTPS)
      - CPU, I/O, Memory 등
    + MMS의 Web UI로 Dashboard제공

## Consul
  * 개요 : https://www.consul.io/intro/index.html
    + Service Discovery : Consul의 client들은 API, MySQL같은 서비스를 제공하고, 서로 DNS나 HTTP 방식을 통해 속해있는 서비스들을 찾음
    + Health Checking : Consul의 client들은 health check를 위한 서비스를 제공하고, 이를 통해 각 node를 모니터링하거나, traffic을 건강한 node로 route시킴
    + KV Store : Application은 HTTP로 Consul의 KV store를 이용해 (Node들의*)동적 설정, flagging, 구성 등 다양한 작업이 가능
    + Multi Datacenter : 다양한 region으로 확장하기 위해 여러개의 datacenter로 구성(좀 더 확인필요)
      - Datacenter : Low latency와 High bandwidth를 갖춘 네트워크망의 서버들
      - 물리적으로 가깝거나, 가상화된 서버들의 묶음을 의미하는 듯 (참조: http://www.moreagile.net/2015/12/Automating-the-Modern-Datacenter-with-Terraform-and-Consul.html)
    + DevOps 친화적으로 설계되었음
  * Architecture
    + 분산 구조의 HA 시스템, 모든 Node들은 health check를 위해 Consul agent를 설치함
      - RPC(Remote Procedure Call) 통신 기반(8300~8302 port를 사용하는 듯, RPC 8300, LAN Gossip 8301, WAN Gossip 8302)
      - Agent는 Server mode / Client mode로 구분되는 daemon, Consul cluster의 모든 node에 설치됨
      - Server : cluster의 상태 관리, datacenter간의 통신, 질의 전달(leader또는 다른 datacenter로) 수행
      - Client : 적은 자원과 데이터 소모
    + Datacenter별로 Consul Server(+Client) cluster를 운영
    + 데이터를 보관하는 하나 이상의 Consul Server, Client들은 Server와 통신
      - 3~5개의 서버로 클러스터를 구축하기를 추천
      - Server들은 Leader를 뽑음
    + Service를 찾기위한 질의는 Server 또는 Client로..(Agent가 Server로 질의 전달함)
      - 사전에 Agent에 Service를 등록해야 함
  * Gossip protocol을 기반으로 구축되어 다른 모니터링 도구 처럼 Central Server에 부하를 주지 않음
    + 지속적인 Polling이 아닌, Node의 state가 변할때만 trigger됨(traffic이 적음)
    + Agent가 이상이 있어도 분산된 detector에 의해 최종적으로 failure를 찾을 수 있음(Edge triggered architecture?)
  * Nagios보다 쉬운 설정으로 Node를 늘이거나 DevOps하기에 친화적임
  * 전체 Node의 상태를 질의해야하는 Chef와 달리 Service Discovery Tool로 디자인 되어 보다 동적이고 민감하게 클러스터를 관리
  * 개인적인 평가 : 진보된 분산 환경의 서버 모니터링/관리 도구, 작은 규모에서는 불필요 할듯


## Note
모니터링 도구 요약 정리  
https://m.blog.naver.com/PostView.nhn?blogId=kmk1030&logNo=220741824042&proxyReferer=https%3A%2F%2Fwww.google.com.tw%2F
