---
title: 离线装 docker-compose
author: 陈龙
tags:
  - docker
categories:
  - 技术
  - docker
keywords:
  - docker
  - 离线
date: 2023-07-27 10:48
---
官方文档：[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

下载地址：[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

```shell
cp docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
chmod 755 /usr/local/bin/docker-compose
 
docker-compose --version

```

## 离线安装docker在step 3之后启动不了docker，就需要添加如下两条

``` shell
chmod +x  /usr/lib/systemd/system/docker.service

systemctl daemon-reload
```

再次启动docker就可以了