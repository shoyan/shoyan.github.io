---
layout: post
title: "Dockerでsystemctlでserviceが起動できない"
date: 2016-04-14 20:10:46 +0900
comments: true
categories: Docker
---
centos7からsystemctlでserviceを起動するようになったが、Dockerで起動すると「Failed to get D-Bus connection: No connection to service manager.」というエラーメッセージがでて起動できないという問題が起こった。


```
$ docker run -it centos:centos7 /bin/bash
# systemctl
Failed to get D-Bus connection: No connection to service manager.

```

サービスを起動するためには以下の方法でコンテナを起動すればよい。

```
$ docker run --privileged -d --name httpd centos:centos7 /sbin/init

```

起動したコンテナでhttpdをinstallして起動する。


```
$ docker exec -it httpd /bin/bash
# yum install httpd -y
# systemctl enable httpd.service
# systemctl start httpd.service

```

プロセスを確認。起動できている。


```
ps aux | grep apache

bash-4.2# ps aux | grep apache
apache     168  0.0  0.3 221912  4028 ?        S    05:41   0:00 /usr/sbin/httpd -DFOREGROUND
apache     169  0.0  0.3 221912  4028 ?        S    05:41   0:00 /usr/sbin/httpd -DFOREGROUND
apache     170  0.0  0.3 221912  4028 ?        S    05:41   0:00 /usr/sbin/httpd -DFOREGROUND
apache     171  0.0  0.3 221912  4028 ?        S    05:41   0:00 /usr/sbin/httpd -DFOREGROUND
apache     172  0.0  0.3 221912  4028 ?        S    05:41   0:00 /usr/sbin/httpd -DFOREGROUND
root       180  0.0  0.0   9044   808 ?        S+   05:49   0:00 grep apache

```

yunanoさんの記事が大変参考になった。  
http://qiita.com/yunano/items/9637ee21a71eba197345
