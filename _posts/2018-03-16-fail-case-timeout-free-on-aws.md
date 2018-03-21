---
layout: post
title:  "금주의 AWS 실패사례 - Resource가 삭제된 CloudFormation stack을 update하다 dead-lock"
date:   2018-03-16 17:00:00 +0900
categories: AWS
comments: true
---
# AWS에서 동작하는 Web Application Server의 Timeout설정


## Local 환경
  * Browser나 PostMan, Eclipse 등에서도 서버로 부터 응답이 없으면 멈추는 Timeout설정이 있으니 반드시 체크할 것

## API Gateway
  * 29~50초가 기본
  * 수정 불가 : https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html#api-gateway-limits
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
    + location 설정 아래에 다음과 같은 설정값 입력 : https://stackoverflow.com/questions/24453388/nginx-reverse-proxy-causing-504-gateway-timeout
    http://annajinee.tistory.com/15
    ~~~ ruby
    proxy_connect_timeout       300;
    proxy_send_timeout          300;
    proxy_read_timeout          300;
    send_timeout                300;
    ~~~
    + 설정 파일의 위치는 다음을 참고 : https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html
    ~~~
    ~/workspace/my-app/
    |-- .ebextensions
    |   `-- nginx
    |       `-- conf.d
    |           `-- myconf.conf
    `-- WEB-INF
    ~~~
  * 그!러!나! 안됨!
    + 이유
      - 참조, 실제 EC2에 설치된 nginx의 설정(nginx.conf)
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
      - .ebextensions/nginx/conf.d/ 파일은 EC2의 /etc/nginx/conf.d 로 복사됨
      - conf.d 폴더의 파일들은 http항목에 include됨
      - 그러나 location항목은 뒤에 나오는 conf.d/elaticbeanstalk 폴더 아래에 있는 00_application.conf 가 다시 한번 include됨
      - 이것으로 인해 사용자가 원한 location 설정은 무시됨(Override되어 버리는 듯)
    + 해결책
      - SSH를 통해 해당 EC2에 접속하여 00_application.conf 파일을 위에서 설명한 방법으로 수정해야함
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
      - conf.d/elasticbeanstalk/ 파일들은 http.server항목 아래에 include되므로 바로 location을 정의하면 됨
      - 아래와 같이 ebextensions를 설정해 보았으나 EC2에 conf파일이 추가되지도 않고, 서비스 재시작시 에러만 발생
      ~~~
      ~/workspace/my-app/
      |-- .ebextensions
      |   `-- nginx
      |       `-- conf.d
      |           `-- elasticbeanstalk
      |               `-- myconf.conf
      `-- WEB-INF
      ~~~
      - 이 방법은 당장에는 동작하겠지만 새로운 war파일을 deploy하면 무효가 되므로 주의
    + nginx.conf를 통째로 override
      - 본문에 첨부한 nginx.conf를 완전히 수정한 후, 다음 path에 두고 deploy하면 동작 함
      ~~~
      ~/workspace/my-app/
      |-- .ebextensions
      |   `-- nginx
      |       `-- nginx.conf
      `-- WEB-INF
      ~~~

## Tomcat server.xml
  * 60000밀리초(주의!)가 기본
  * Connector connectionTimeout="60000" 부분을 바꾸면 된다고 하나 엄밀히 말하면 다른 개념이라고 함
    + 연결 생성을 기다리는 시간일 뿐: https://okky.kr/article/257636
    + 다음처럼 timeout을 강화 하는 방법도 존재(기본 10분 설정) : https://stackoverflow.com/questions/7145131/tomcat-request-timeout
  * Connector keepAliveTimeout="60000" 일까?
    + connectionTimeout 값을 기본으로 가짐. 즉, connectionTimeout을 수정하면 되긴 함 : https://tomcat.apache.org/tomcat-7.0-doc/config/http.html#Standard_Implementation
  * 이렇게 바꾼 것을 서버에 적용하기 위해서는 추가적인 ebextensions 파일이 필요
    + 수정한 server.xml을 .ebextensions 폴더에 추가
    + .ebextensions 폴더 아래에 다음 내용의 server-update.config 파일을 생성(Tomcat 버전 주의)
    ~~~ yaml
      container_commands:
        replace-config:
          command: cp .ebextensions/server.xml /etc/tomcat8/server.xml
    ~~~

## Application(Spring Framework)
  * 여기도 뭔가 있지만 생략
