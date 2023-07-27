---
title: docker离线安装
author: 陈龙
tags:
  - docker
categories:
  - 技术
  - docker
keywords:
  - docker
  - 离线
date: 2023-07-27 10:35
---
## 下载docker离线包
[docker离线包下载](https://download.docker.com/linux/static/stable/x86_64/)
## 准备docker.service 系统配置文件(docker启动关闭都靠它)
- docker.service
``` shell
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
 
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target
```
## 安装与卸载脚本
1. 安装脚本
 - install.sh 
``` shell
#!/bin/sh
echo '解压tar包...'
tar -xvf $1
echo '将docker目录移到/usr/bin目录下...'
cp docker/* /usr/bin/
echo '将docker.service 移到/etc/systemd/system/ 目录...'
cp docker.service /etc/systemd/system/
echo '添加文件权限...'
chmod +x /etc/systemd/system/docker.service
echo '重新加载配置文件...'
systemctl daemon-reload
echo '启动docker...'
systemctl start docker
echo '设置开机自启...'
systemctl enable docker.service
echo 'docker安装成功...'
docker -v
```

2. 卸载脚本
   - uninstall.sh
``` shell
#!/bin/sh
echo '删除docker.service...'
rm -f /etc/systemd/system/docker.service
echo '删除docker文件...'
rm -rf /usr/bin/docker*
echo '重新加载配置文件'
systemctl daemon-reload
echo '卸载成功...'

```
## 安装与卸载
- 此时目录有:
  ` docker-18.03.0-ce.tgz、docker.service、install.sh、uninstall.sh `
- 执行脚本 ` sh install.sh docker-18.03.0-ce.tgz ` 如果你想卸载docker，此时执行脚本` sh uninstall.sh ` 即可
## 配置加速源

``` shell
vim /etc/docker/ daemon.json 加入
```

``` shell
{
"registry-mirrors": [
"https://kfwkfulq.mirror.aliyuncs.com",
"https://2lqq34jg.mirror.aliyuncs.com",
"https://pee6w651.mirror.aliyuncs.com",
"https://registry.docker-cn.com",
"http://hub-mirror.c.163.com"
],
"dns": ["8.8.8.8","8.8.4.4"]
}
```

``` shell
systemctl daemon-reload 加载  
systemctl restart docker 重启
```
