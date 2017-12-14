---
layout: post
title:  "금주의 실패사례 - Apache 서버의 Basic Authentication으로 LDAP 적용하기"
date:   2017-12-13 16:00:00 +0900
categories: Linux Ubuntu Web LDAP Authentication
comments: true
---
# 이번 주 Apache를 사용하면서 겪은 실패 사례
보안을 위해 web page에 LDAP 인증을 적용해 보자  
LDAP을 적용하기 위해서는 도대체 뭘 수정해야 한단말인가?!

## 개요
  * 모니터링 도구의 웹 dashboard나 간단한 웹 페이지를 운영할 경우 특정 사용자만 해당 페이지로 접근하게 하고 싶음
  * Apache 서버 기반
  * Basic Authentication
    + 웹 페이지 로딩 시 Web Server에 Basic Authentication을 설정하면 제한된 사용자만 접속 할 수 있음
      - 아주 간단한 로그인 dialog box가 표시됨
    + 보통 Web Server의 conf 파일이나, 설치된 모니터링 도구의 conf파일을 수정하면 됨
      - 설치된 모니터링 도구의 conf파일을 symbolic link로 apache의 conf로 읽어옴(overriding 개념인듯*)

## 설정 위치
  * Ubuntu의 경우 /etc/apache2
    + /etc/apache2/apache2.conf
    + /etc/apache2/conf-enabled/도구.conf
    + /etc/apache2/conf-available/도구.conf
  * AmazonLinux AMI의 경우 /etc/httpd
    + 이유 : 기존에는 httpd(Daemon)였으나 ubuntu에서는 전반적인 구조가 바뀜 [[참고: Ubuntu 도움말](https://help.ubuntu.com/lts/serverguide/httpd.html)]
  * 모니터링 도구 설정인 경우 /etc/도구
    + 도구별로 설치 위치가 다를 수 있음

## LDAP 동작 개요
  * LDAP에 대한 설명은 생략
  * LDAP을 통해 인증 받으려면 LDAP서버의 AuthLDAPUrl, AuthLDAPBindDN와 AuthLDAPBindPassword을 이용하여 인증 요청을 해야함
    + Require ldap-group으로 특정 그룹의 사용자만 Authorization할 수 있음
    + 예제
~~~
    AuthName "Restricted Area"
    AuthType Basic
    AuthBasicProvider ldap
    AuthLDAPUrl "ldap://10.0.0.1:10389/dc=iam,dc=aws,dc=org?uid?sub?(objectclass=posixaccount)"
    AuthLDAPBindDN "uid=admin,ou=system"
    AuthLDAPBindPassword "password"
    Require ldap-group cn=AdminGroup,ou=groups,dc=iam,dc=aws,dc=org
    AuthLDAPGroupAttributeIsDN off
    AuthLDAPGroupAttribute memberUid
~~~
    + Require의 cn에 따옴표를 붙이지 말 것
    + 참고 [[Apache httpd](https://httpd.apache.org/docs/2.4/mod/mod_authnz_ldap.html)]

## 설정 후
  * 수정 후 서버 재시작 : sudo service apache2 restart

## LDAP말고 local사용자로 로그인
  * htaccess로 로그인하기
    + LDAP외에 local drive에 파일로 사용자 정보를 관리하고 싶다면 htaccess를 설정하면 됨
    + 설정 방법은 LDAP과 유사
      - htaccess파일은 AuthUserFile에서 경로를 지정
    + Web Server에서 LDAP으로 할지 htaccess로 할지 결정은 /etc/apache2/conf-enabled/에 위치한 conf파일을 참조
    + conf파일 내 <Location>의 웹 페이지 경로에 해당할 경우 적용됨


    ~~~ ddd
    <Location /munin/>
            Order allow,deny
            Allow from All
             AuthName "Restricted Area"
             AuthType Basic
             AuthUserFile /etc/apache2/htpasswd
             Require valid-user
    </Location>
    ~~~

  * htaccess파일에 암호를 지정 방법
    + htpasswd 파일 경로는 Basic Authentication에 연결되어야 함  
~~~ sh
  $ htpasswd -c /etc/apache2/htpasswd rbowen
    New password: mypassword
    Re-type new password: mypassword
    Adding password for user rbowen
~~~
    + 참조 [[Apache httpd](https://httpd.apache.org/docs/2.4/howto/auth.html)]
