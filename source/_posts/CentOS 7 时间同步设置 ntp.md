---
title: CentOS 7 时间同步设置 ntp
author: 陈龙
tags:
  - linux
  - NTP
categories:
  - linux
  - CentOS
keywords:
  - linux
  - ntp
date: 2023-07-13 13:58
---
一. 安装ntp服务

查看是否安装ntp
``` shell
rpm -qa | grep ntp
```

显示如下：
python-ntplib-0.3.2-1.el7.noarch
fontpackages-filesystem-1.44-8.el7.noarch
ntp-4.2.6p5-29.el7.centos.2.x86_64
ntpdate-4.2.6p5-29.el7.centos.2.x86_64

说明已经安装了ntp服务，如果没有，如果离线安装，下载rpm相关包，进行安装

在 https://centos.pkgs.org/7/centos-x86_64/ 下载

三个包：

autogen-libopts-5.18-5.el7.x86_64.rpm

ntpdate-4.2.6p5-29.el7.centos.2.x86_64.rpm

ntp-4.2.6p5-29.el7.centos.2.x86_64.rpm

依次安装：rpm -ivh XXXXX

也可以使用以下命令在线安装：
``` shell
yum -y install ntp
```

安装完结果如下：
![](656c8fee1972b13ea46a12da5ec44a3f_MD5.png)

## 二、修改配置文件

安装完成后，会在`/etc/`目录下生成配置文件`ntp.conf`

安装完成后，设置配置文件：
``` shell
vim /etc/ntp.conf
```
1.使用#注释掉以下条目
``` shell
server ntp.alibaba.com
 
#server 0.centos.pool.ntp.org iburst
 
#server 1.centos.pool.ntp.org iburst
 
#server 2.centos.pool.ntp.org iburst
 
#server 3.centos.pool.ntp.org iburst

```

2.添加以下行进配置文件：
``` shell
#据说127.127.1.0为本地时钟，待验证
server 127.127.1.0
Fudge 127.127.1.0 stratum 10
```

3.添加以下行进配置文件（ip处替换成本机ip，即内网ntp服务器所在ip）：
``` shell
#此条意为允许该网段所有服务器连接本机获取时间，但禁止修改本机时间
restrict 10.209.22.0 mask 255.255.255.0 nomodify notrap
#允许此ip修改本机时间
restrict 10.209.22.160
```

## 三、启动ntp服务器，并设置开机自启
``` shell
#启动
systemctl start ntpd
#设置开机自启
systemctl enable ntpd
#查看服务状态
systemctl status ntpd
```

## 四、配置其他服务器ntp时间同步

1.使用#注释掉以下条目：
``` shell
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
```

2.添加以下条目进配置文件：
``` shell
#允许此ip修改本机时间
restrict 10.209.22.160
#获取时间ntp服务器ip
server 10.209.22.160
Fudge 10.209.22.160 stratum 10
```
![](eb16d422a89bb25c36f257154fccedae_MD5.png)

## 五、查看ntp时间同步状态

1.使用`ntpq -p`查看时间同步状态：
![](ccf465fb66bccefffa770b5a02423577_MD5.png)
remote：远程ntp服务器ip或主机名，带“*”的表示本地NTP服务器与该服务器同步  
refid：远程NTP服务器使用的上一级ntp服务器的IP地址  
st：ntp服务器所在stratum阶层  
t：本地NTP与远程NTP服务器的通信方式。u：单播；b：广播；I：本地  
when：上次成功同步时间距现在有多少秒  
poll：本地NTP与远程NTP服务器同步的时间间隔。  
reach：这是一个八进制的值，用来测试衡量前八次查询是否成功和服务器连接。377表示都成功，0表示不成功  
delay：网络延迟  
offset：本地NTP与远程NTP服务器的时间偏移  
jitter：最近两次有变化的offset差的绝对值

2.使用`ntpstat`查看ntp时间同步状态，刚配置完成需要十五分钟左右等待才会出现如下结果，表示ntp服务正常：
![](193c4c1d187febf763b896a30a9061fa_MD5.png)
