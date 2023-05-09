---
title: Centos7开放及查看端口
author: 陈龙
tags:
  - linux
keywords:
  - linux
  - centos
date: 2023-05-09
---
1、开放端口

firewall-cmd --zone=public --add-port=5672/tcp --permanent   # 开放5672端口

firewall-cmd --zone=public --remove-port=5672/tcp --permanent  #关闭5672端口

firewall-cmd --reload   # 配置立即生效

2、查看防火墙所有开放的端口

firewall-cmd --zone=public --list-ports

3.、关闭防火墙

如果要开放的端口太多，嫌麻烦，可以关闭防火墙，安全性自行评估

systemctl stop firewalld.service

4、查看防火墙状态

 firewall-cmd --state

5、查看监听的端口

netstat -lnpt

![0](bab49f1b0d38565213b7dae52f9a16c8_MD5.png)

PS:centos7默认没有 netstat 命令，需要安装 net-tools 工具，yum install -y net-tools

6、检查端口被哪个进程占用

netstat -lnpt |grep 5672

![0](3c8ae59cd93099060c7c10ff9b35c9d5_MD5.png)

7、查看进程的详细信息

ps 6832

![0](14be86afb1183986d5bdcc6373b68160_MD5.png)

8、中止进程

kill -9 6832