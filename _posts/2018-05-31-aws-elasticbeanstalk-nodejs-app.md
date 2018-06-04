---
layout: post
title:  "AWS ElasticBeanstalk에 Nodejs app을 올려보자"
date:   2018-05-31 22:00:00 +0900
categories: AWS Nodejs ElasticBeanstalk
comments: true
---
# AWS ElasticBeanstalk에 Nodejs app을 올려보자
ElasticBeanstalk(이하 EB)는 Web app을 배포하기 딱 좋게 사전 정의된 CloudFormation이다.  
EB가 지원하는 Web Application Server(WAS)는 여러가지가 있고, 주로 Tomcat을 써왔지만, 이번 과제에서는 불가피하게 유료 솔루션에서 제공하는 Nodejs 서비스를 사용해야 하는 상황이 됐다.  
EB에 Nodejs app을 배포하며 겪은 문제를 정리하고자 한다.

## EB를 쓰는 이유
  * 수동 구성의 불편함
    + 수동으로 AWS infra를 구축하면 강력하며, 세세한 설정도 가능
    + 필요한 App을 AMI이미지로 생성하고 CloudFormation으로 자동 배치되도록 설정하면 좋음
    + 그러나 Load Balancer를 포함한 infra를 구축하기 위해선 설정할 것이 너무 많음
      - EC2 image
      - Auto Scaling Group
      - Load Balancer
      - Route Table, Internet Gateway, NAT Gateway 등
  * EB를 쓰면 좋은 점
    + EB의 경우 console로 간편하게 생성가능
    + CloudFormation과 ebextensions를 이용하면 충분히 세세한 설정도 가능
    + Application만 신경쓰면 됨
    + (개인적으로)평소에 자주 써왔기에 익숙함

## EB에 배치할 만한 Nodejs Application
  * AWS Document에선 Express 모듈을 권장함
    + [Node.js 개발 환경 설정](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/nodejs-devenv.html)
  * 이 글을 쓰게된 배경은, 본인의 과제가 Express 모듈을 이용한 것이 아니기 때문
    + Express 모듈은 보편적인 Request/Response 서버용
    + EB Environment에 Worker environment라는게 존재하지만, 이는 진짜 scheduled job에 적합
    + Express 모듈이 없는 app을 Nodejs에서 동작하게 하려면 $ node app.js 처럼 직접 실행시켜야 함
  * EB의 Nodejs App의 설치
    + EB는 로딩 시 무조건 사용자의 설치파일(zip)을 읽어 app을 설치함
      - npm install 수행
      - package.json과 npm-shrinkwrap.json을 참조
      - 참고: [Package.json 파일로 패키지 설치](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/nodejs-platform-packagejson.html)
  * PM2를 이용해 process 수행
    + PM2를 사용하는 이유
      - Nodejs app이 계속 수행되는(멈추지 않는) process라면, 이를 daemon화 시켜서 수행해야함
      - PM2는 Nodejs app을 daemon화 시켜 관리해주는 서비스(아마 다른 서비스도 있을지도?)
      - daenon화 시키지 않으면, EB Environment의 health가 Green이 되지 않고 계속 대기중 상태가 됨
      - 대기중 상태일 경우 EB 설정이 아무것도 활성화 되지 않음
    + PM2를 사용하여 background에서 수행
    ~~~
      $ pm2 start /var/app/current/yourapp -- start -p param
    ~~~
    + PM2에서 권장하는 방법: package.json 파일에 다음 script를 추가
    ~~~ json
      "scripts": {
        "start": "node ./node_modules/pm2/bin/pm2 start app.js --name yourApp",
        "poststart":  "node ./node_modules/pm2/bin/pm2 logs"
      }
    ~~~
      - 위와 같이 할 경우, Node Command에서 알아서 pm2를 이용한 app실행이 된다고 함(안해봤음)
      - 참고: [Using PM2/Keymetrics in AWS Elastic Beanstalk](http://pm2.keymetrics.io/docs/tutorials/use-pm2-with-aws-elastic-beanstalk)

## ebextensions를 이용한 후처리 추가
EB는 ebextensions를 통해 추가 작업이 가능, 그러나 보통은 Application 로드 전까지 환경을 구축하는 역할로 활용  
EB의 appdeploy/post 폴더에 스크립트를 넣어두면, Application 로드 후에 작업이 진행됨
  * Nodejs환경에서 작업 실행
    + EB는 기본적으로 Software설정에 Node Command라는 configurations를 전달 받을 수 있음
    + 기본 값은 node app.js, node server.js, npm start 가 순차적으로 실행 됨
    + 위 값은 단 한 줄의 command만 전달되므로, 복잡한 command의 경우 후처리를 이용해야 함
  * 설정 방법
    + 참고: [Execute command after deploy AWS Beanstalk](https://stackoverflow.com/a/28866602)
    + 그러나 node나 npm이 환경변수로 path설정이 되어 있지 않아 그냥 실행시 에러 발생(ex: "npm: command not found")
      - 위 참고의 댓글을 참고
      ~~~
        $ PATH=$PATH:"$(ls -td /opt/elasticbeanstalk/node-install/node-*/bin | head -1)"
      ~~~
      - 환경변수 path에 node폴더가 지정되어 있어야 node나 npm, npm으로 설치된 서비스 들이 호출 가능해짐
  * 위 작업을 한 이후 node, npm 명령 수행 가능
    + script 예제
    ~~~
    commands:
      create_post_dir:
        command: "mkdir /opt/elasticbeanstalk/hooks/appdeploy/post"
        ignoreErrors: true
    files:
      "/opt/elasticbeanstalk/hooks/appdeploy/post/run_postjob.sh":
        mode: "000755"
        owner: root
        group: root
        content: |
          #!/usr/bin/env bash
          PATH=$PATH:"$(ls -td /opt/elasticbeanstalk/node-install/node-*/bin | head -1)"
          npm install -g pm2
          pm2 start /var/app/current/yourapp -- start -p param
    ~~~
      - /var/app/current/폴더는 사용자가 업로드한 zip파일이 복사된 폴더

## root 계정으로 보기
참고로 ElasticBeanstalk에서 console로 받은 명령은 EC2의 ec2-user계정이 아닌 root계정으로 수행됨  
따라서 EC2의 내용을 확인하기 위해 root 계정으로 접근할 방법이 필요할 수 있음
  * Root 계정 접근 방법
  ~~~
    $ sudo -i
  ~~~
    + 참고: [How do I ssh as root](https://forums.aws.amazon.com/message.jspa?messageID=318334#jive-message-318321)
