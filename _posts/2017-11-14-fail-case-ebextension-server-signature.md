---
layout: post
title:  "금주의 AWS 실패사례 - ElasticBeanstalk의 Server Signature 변경"
date:   2017-11-14 16:00:00 +0900
categories: AWS ElasticBeanstalk Web EC2 Failure
comments: true
---
# 이번 주 AWS를 사용하면서 겪은 실패 사례
보안을 위해 server response의 header중 server signature를 변경해보자

## 개요
  * REST API나 Web Application의 경우 HTTP응답에는 Header가 존재
  * Header에는 여러 정보가 포함되어 있으며, 그 중  Server Signature는 서버의 정보를 알려줌
    + Server: nginx/1.10.2
    + Server: Apache-Coyote/1.1
    + 위 처럼 서버의 종류와 버전이 표시됨
  * 이는 해당 서버의 보안 취약점을 알아내기 쉽게 만들기 때문에 보안상 숨기는게 좋음
  * EC2는 ssh로 접근하여 수정
  * ElasticBeanstalk를 통해 생성되는 EC2는 어떻게 수정 할 수 있을까?

## nginx의 Server Signature 숨기는 법
  * Google에서 검색해보면 다양하게 안내하고 있음 : nginx server signature off
    + [1번](https://stackoverflow.com/questions/24594971/how-to-changehide-the-nginx-server-signature)
    + [2번](https://serverfault.com/questions/214242/can-i-hide-all-server-os-info)
    + [3번](https://talk.plesk.com/threads/how-to-disable-nginx-server-signature.340216/)
  * 위 링크들을 정리해보면
    + /etc/nginx/nginx.conf 파일을 찾아서 http 아래에
      - server_tokens off; 를 추가
      - 이것으로 서버 버전을 숨길 수 있음
    + 추가로 서버종류를 숨기거나 바꿀 수 도 있는데
      - more_set_headers 'Server: NONE-OF-YOUR-BUISINESS'; 로 서버 이름을 변경
      - more_clear_headers 'Server'; 로 서버 정보 자체를 삭제
      - 이 부분은 안된다는 글도 본 것 같음
    + 그리고 서버 서비스를 재시작

## ElasticBeanstalk에서 nginx 설정 변경
  * ElasticBeanstalk는 reserved proxy로 nginx를 지원
  * .ebextension 폴더에 설정 파일을 넣어두면 Deployment시 서버에 반영함
  * [공식 가이드](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/java-se-nginx.html)([한글](http://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/java-se-platform.html#java-se-nginx))에서 다음과 같이 안내
    + .ebextensions/nginx/nginx.conf 을 추가하면 구성을 완전히 재정의 함
      - 처음엔 nginx.conf 파일에서 설정 일부만 작성하면 해당 설정을 override될 줄 알았는데 아니었음
      - 다음과 같은 에러 발생

~~~
[2017-11-13T04:55:21.342Z] ERROR [5197]  : Command execution failed: Activity failed. (ElasticBeanstalk::ActivityFatalError)
caused by: Executing: /opt/elasticbeanstalk/bin/log-conf -n nginx -l'/var/log/nginx/*'

Nginx configuration detected in the '.ebextensions/nginx' directory. AWS Elastic Beanstalk will no longer manage the Nginx configuration for this environment.
Executing: /usr/sbin/nginx -t -c /var/elasticbeanstalk/staging/nginx/nginx.conf
nginx: [emerg] unknown directive "more_set_headers" in /var/elasticbeanstalk/staging/nginx/nginx.conf:2
nginx: configuration file /var/elasticbeanstalk/staging/nginx/nginx.conf test failed
Failed to execute '/usr/sbin/nginx -t -c /var/elasticbeanstalk/staging/nginx/nginx.conf'
Failed to execute '/usr/sbin/nginx -t -c /var/elasticbeanstalk/staging/nginx/nginx.conf' (ElasticBeanstalk::ExternalInvocationError)
caused by: Executing: /opt/elasticbeanstalk/bin/log-conf -n nginx -l'/var/log/nginx/*'
~~~
      - 아마 nginx.conf 파일의 구성이 완벽하지 못해 load가 되지 않는 다는 뜻인듯
    + .ebextensions/nginx/conf.d/myconf.conf 을 추가하면 해당 설정을 override함
      - [AWS Blog](https://aws.amazon.com/blogs/aws/elastic-beanstalk-update-support-for-java-and-go/) 설명 참조
      - 여기서 http 아래에 server_tokens off;를 입력할 필요 없이, 바로 본문에 작성하면 됨
      - X: http { server_tokens off; }
      - O: server_tokens off;

## Apache Tomcat의 Server Signature 숨기는 법
  * Google에서 검색해보면 정말 많은 글을 볼 수 있음
  * 다음이 제일 명료한듯
    + [Turn off server signature on Apache web server](https://blog.oshim.net/2016/04/turn-off-server-signature-on-apache-web/)
    + [How to turn off server signature on Apache web server](http://ask.xmodulo.com/turn-off-server-signature-apache-web-server.html)
      - 여기서 설명한대로 ServerSignature Off 는 에러 페이지에서 서버 정보를 숨기는 것
      - ServerTokens Prod로 하면 Response의 header에서 서버 정보가 숨겨짐
  * httpd.conf 파일은 어디에 있을까?
    + Linux에 익숙하지 않아 파일 [검색방법1](https://serverfault.com/questions/49879/cant-find-httpd-conf), [검색방법2](https://www.digitalocean.com/community/tutorials/how-to-use-find-and-locate-to-search-for-files-on-a-linux-vps) 찾기
    + 힌트 발견, [Apache설정 말고 Tomcat설정을 바꾸라는 글](https://stackoverflow.com/questions/10928516/unable-to-find-httpd-conf)
    + conf 파일을 수정
  * 그리고 [서버 서비스(service httpd) 재시작](https://stackoverflow.com/questions/4062723/restart-httpd-after-changes-in-the-httpd-conf)
  * 왠지 모르겠지만 잘 안됨... 추가 확인 필요

## ElasticBeanstalk에서 Apache 설정 변경
  * nginx와 달리 Apache는 ebextension설정이 훨씬 복잡
    + config라는 확장자를 가진 YAML형식의 파일로 설정 수행
    + files로 필요한 파일들을 EC2로 복사(또는 생성)
    + container_commands로 EC2에서 shell script 수행    
  * 오래된 글(2012년)이지만 [Amazon Linux에서 동작하지 않는다는 글](https://forums.aws.amazon.com/thread.jspa?messageID=348194#348194)도 존재
  * ElasticBeasntalk가 [Virtual host를 생성하기 떄문](https://forums.aws.amazon.com/thread.jspa?messageID=770295)이라는 설명
    + .ebextensions/httpd/conf.d/elasticbeanstalk 폴더 아래에 00_application.conf 파일을 만들어 설정해야 한다고 함
    + 전반적으로 너무 복잡해지기 시작해서 어려움...
  * [Tomcat설정을 변경하면 간단하다는 글](https://stackoverflow.com/questions/11102511/remove-server-header-tomcat) 발견
    + Response header에서 Server를 삭제할 수 없음
    + Tomcat의 server.xml에서 Connector에 server property를 입력하면 그걸로 표시됨
  * .ebextensions/에서 config파일을 생성
~~~ yaml
  container_commands:
      replace-config:
        command: cp .ebextensions/server.xml /etc/tomcat7/server.xml
~~~
    + shell script로 server.xml파일을 덮어쓰기

## Server Signature를 숨긴 것의 의미
  이렇게 숨긴다고 하더라도 다양한 방법(에러 페이지 호출 등)으로 서버정보를 알아 낼 수 있기 때문에, 실질적인 보안강화 요소가 아님.  
  필요한 port만 개방하고, 서버 보안 패치를 하는게 중요할 듯

## 기타 참고사항
  + Private Subnet의 EC2에 붙기  
https://aws.amazon.com/ko/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/
  + Putty로 EC2에 붙기  
https://linuxacademy.com/howtoguides/posts/show/topic/17385-use-putty-to-access-ec2-linux-instances-via-ssh-from-windows  
https://cpuu.postype.com/post/30065
