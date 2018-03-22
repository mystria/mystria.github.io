---
layout: post
title:  "금주의 AWS 실패사례 - AWS에서 동작하는 Web Application Server의 Timeout설정"
date:   2018-03-16 17:00:00 +0900
categories: AWS ElasticBeanstalk
comments: true
---
# AWS에서 동작하는 Web Application Server의 Timeout설정


## Local 환경
* Browser나 PostMan, Eclipse 등에서도 서버로 부터 응답이 없으면 멈추는 Timeout설정이 있으니 반드시 체크할 것

## API Gateway
* 29~50초가 기본
* 수정 불가: [AmazonAWS Docs]( https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html#api-gateway-limits)  

  ~~~
  504 Gateway Timeout

  {
    "message": "Endpoint request timed out"
  }
  ~~~

## Load Balancer
* 60초가 기본
* 응답 헤더에 특별한 내용 없음

  ~~~
  504 GATEWAY_TIMEOUT

  Cache-Control →proxy-revalidate
  Connection →Keep-Alive
  Content-Length →0
  Date →Fri, 16 Mar 2018 10:23:28 GMT
  ~~~
* Load Balancer의 Description > Attributes > Idle timeout 수정

## Nginx reverse proxy
* 60초가 기본
* 응답 해더 정보의 Server항목에 nginx/1.x.x(버전) 라고 표시됨(서버 정보를 일부러 숨긴 경우는 안 나올 수 있음)
  + 깨알 Tip : server_tokens off; 를 ebextensions를 이용해 삽입하면 서버 버전이 보이지 않음
* ebextensions을 이용해 nginx 서버 설정 변경
  + location 설정 아래에 다음과 같은 설정값 입력: [StackOverflow: Nginx reverse proxy causing 504 Gateway Timeout]( https://stackoverflow.com/questions/24453388/nginx-reverse-proxy-causing-504-gateway-timeout)  
   [Proxy설정 및 기본세팅](http://annajinee.tistory.com/15)
  ~~~ ruby
    proxy_connect_timeout       300;
    proxy_send_timeout          300;
    proxy_read_timeout          300;
    send_timeout                300;
  ~~~
  + "location /" 설정은 http항목 아래의 server 항목에 삽입되어야 하므로 elasticbeanstalk 폴더 아래에 넣자
  + 설정 파일자의 위치는 다음을 참고: [AmazonAWS Docs](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-tomcat-proxy.html#java-tomcat-proxy-nginx)
  ~~~
    ~/workspace/my-app/
    |-- .ebextensions
    |   `-- nginx
    |       `-- conf.d
    |           |-- elasticbeanstalk
    |           |   `-- 00_application.conf
    |           |   `-- 01_static.conf
    |           |   `-- my-server-conf.conf
    |           `-- my-http-conf.conf
    `-- WEB-INF
  ~~~
* 주의 사항
  + "location /" 설정은 이미 00_application.conf에 되어 있으므로, 다른 이름의 .conf파일로 만들면 중복오류 발생
  ~~~
  [2018-03-16T03:40:13.046Z] ERROR [10436] : Command execution failed: Activity failed. (ElasticBeanstalk::ActivityFatalError)
    caused by: Executing: /opt/elasticbeanstalk/bin/log-conf -n nginx -l'/var/log/nginx/*'

    Executing: /usr/sbin/nginx -t -c /var/elasticbeanstalk/staging/nginx/nginx.conf
    nginx: [emerg] duplicate location "/" in /var/elasticbeanstalk/staging/nginx/conf.d/elasticbeanstalk/02_timeout.conf:1
    nginx: configuration file /var/elasticbeanstalk/staging/nginx/nginx.conf test failed
    Failed to execute '/usr/sbin/nginx -t -c /var/elasticbeanstalk/staging/nginx/nginx.conf'
    Failed to execute '/usr/sbin/nginx -t -c /var/elasticbeanstalk/staging/nginx/nginx.conf' (ElasticBeanstalk::ExternalInvocationError)
    caused by: Executing: /opt/elasticbeanstalk/bin/log-conf -n nginx -l'/var/log/nginx/*'
  ~~~
  + 00_application.conf 파일을 덮어 쓰도록 하자!
  ~~~ ruby
    location / {
        proxy_pass          http://127.0.0.1:8080;
        proxy_http_version  1.1;

        proxy_set_header    Connection          $connection_upgrade;
        proxy_set_header    Upgrade             $http_upgrade;
        proxy_set_header    Host                $host;
        proxy_set_header    X-Real-IP           $remote_addr;
        proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
        proxy_connect_timeout       300;
        proxy_send_timeout          300;
        proxy_read_timeout          300;
        send_timeout                300;
    }
  ~~~
* 참고사항
  + 실제 EC2에 설치된 nginx의 설정(nginx.conf), 2018년 03월 버전, 향후 업데이트될 수 있음
  ~~~ ruby
    # Elastic Beanstalk Nginx Configuration File
      user                    nginx;
      error_log               /var/log/nginx/error.log warn;
      pid                     /var/run/nginx.pid;
      worker_processes        auto;
      worker_rlimit_nofile    19448;

      events {
          worker_connections  1024;
      }

      http {
          include       /etc/nginx/mime.types;
          default_type  application/octet-stream;

          log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" "$http_x_forwarded_for"';

          include       conf.d/*.conf;

          map $http_upgrade $connection_upgrade {
              default     "upgrade";
          }

          server {
              listen        80 default_server;
              access_log    /var/log/nginx/access.log main;

              client_header_timeout 60;
              client_body_timeout   60;
              keepalive_timeout     60;
              gzip                  on;
              gzip_comp_level       4;
              gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

              # Include the Elastic Beanstalk generated locations
              include conf.d/elasticbeanstalk/*.conf;
          }
      }
  ~~~
  + .ebextensions/nginx/conf.d/ 파일은 EC2의 /etc/nginx/conf.d 로 복사됨
    - conf.d 폴더의 파일들은 http항목에 include됨
  + .ebextensions/nginx/conf.d/elasticbeanstalk/ 파일은 EC2의 /etc/nginx/conf.d/elasticbeanstalk 로 복사됨
    - elaticbeanstalk 폴더의 파일들은 server항목에 include됨
    - location 설정은 server아래에 있으므로 이 폴더에 넣어야 함
    - 설정이 겹치거나 문법이 틀리면 서버 로딩시 에러 발생 (logs의 /var/log/eb-commandprocessor.log항목)
  + nginx.conf를 통째로 override하는 방법도 존재
    - 본문에 첨부한 nginx.conf를 완전히 수정한 후, 다음 path에 두고 deploy
    ~~~
      ~/workspace/my-app/
      |-- .ebextensions
      |   `-- nginx
      |       `-- nginx.conf
      `-- WEB-INF
    ~~~
  + 참고: [AmazonAWS Docs](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html)

## Tomcat server.xml
* 60000밀리초(주의!)가 기본
* Connector connectionTimeout="60000" 부분을 바꾸면 된다고 하나 엄밀히 말하면 다른 개념이라고 함
  + 연결 생성을 기다리는 시간일 뿐: [OKKY: tomcat service timeout 설정](https://okky.kr/article/257636)
  + 다음처럼 timeout을 강화 하는 방법도 존재(기본 10분 설정): [StackOverflow: Tomcat request timeout]( https://stackoverflow.com/questions/7145131/tomcat-request-timeout)
* Connector keepAliveTimeout="60000" 일까?
  + connectionTimeout 값을 기본으로 가짐. 즉, connectionTimeout을 수정하면 되긴 함: [Tomcat Docs]( https://tomcat.apache.org/tomcat-7.0-doc/config/http.html#Standard_Implementation)
* 이렇게 바꾼 것을 서버에 적용하기 위해서는 추가적인 ebextensions 파일이 필요
  + 수정한 server.xml을 .ebextensions 폴더에 추가
  + .ebextensions 폴더 아래에 다음 내용의 server-update.config 파일을 생성(Tomcat 버전 주의)
  ~~~ yaml
    container_commands:
      replace-config:
        command: cp .ebextensions/server.xml /etc/tomcat8/server.xml
  ~~~
  + 참고: [StackOverflow: How do I supply configuration to elastic beanstalk tomcat]( https://stackoverflow.com/questions/12264432/how-do-i-supply-configuration-to-elastic-beanstalk-tomcat)
* 그!러!나! 안됨!!
  + 이유: Tomcat이 알아서 해결하므로 설정을 바꿔도 소용없다.
    - [Coderanch: Tomcat timeout](https://coderanch.com/t/663316/application-servers/Tomcat-timeout)
    - [StackOverflow: tomcat request timeout](https://stackoverflow.com/a/7418926/8350542)
  + 해결책
    - Tomcat에서는 timeout을 안심하고 무시하고 nginx proxy나 LoadBalancer 등에서 처리하자..(?)

## Application(Spring Framework)
* 여기도 뭔가 있지만 생략
