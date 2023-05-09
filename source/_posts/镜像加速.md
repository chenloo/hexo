---
title: 镜像加速
author: 陈龙
categories:
  - 技术
  - docker
tags:
  - docker
keywords:
  - docker
abbrlink: 63033431
date: 2023-05-09 00:00:00
---
## 编辑docker配置文件
```shell 
vi /etc/docker/daemon.json
```
## 设置镜像
阿里云
登录：https://cr.console.aliyun.com/cn-qingdao/instances/mirrors
https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
复制加速地址
```json
{
  "registry-mirrors": ["https://wh5u3sdx.mirror.aliyuncs.com"]
}
{
  "registry-mirrors": ["https://3zlp99cx.mirror.aliyuncs.com"]
}
```
网易云
```json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

## 重启docker
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```