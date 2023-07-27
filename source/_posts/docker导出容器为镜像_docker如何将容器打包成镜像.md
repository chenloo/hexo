---
title: docker导出容器为镜像_docker如何将容器打包成镜像
author: 陈龙
tags:
  - docker
categories:
  - 技术
  - docker
keywords:
  - docker
date: 2023-07-13 14:22
---
1、docker镜像、容器导出方式
``` shell
docker save #ID or #Name 
docker export #ID or #Name
```

2、save和export区别 
（1）对于Docker Save方法，会保存该镜像的所有历史记录 
（2）对于Docker Export 方法，不会保留历史记录，即没有commit历史 
（3）docker save保存的是镜像（image），docker export保存的是容器（container）； 
（4）docker load用来载入镜像包，docker import用来载入容器包，但两者都会恢复为镜像； 
（5）docker load不能对载入的镜像重命名，而docker import可以为镜像指定新名称。

3、save命令 
docker save [options] images [images…] 
示例：
docker save -o nginx.tar nginx:latest 
或 
docker save > nginx.tar nginx:latest 
其中 -o 和 > 表示输出到文件，nginx.tar为目标文件，nginx:latest是源镜像（name:tag）

4、load命令 
``` shell
docker load [options] 
```
示例： 
``` shell
docker load -i nginx.tar 
```
或 
``` shell
docker load < nginx.tar 
```
其中 -i 和 < 表示从文件输入。会成功导入镜像及相关元数据，包括tag信息

5、export命令
``` shell
docker export [options] container
```

示例: 
``` shell
docker export -o nginx-test.tar nginx-test
```

 # 导出为tar
``` shell
docker export # ID or #name > /home/export.tar
```

其中-o表示输出到文件，nginx-test.tar为目标文件，nginx-test是源容器名（name）

6、import命令 
``` shell
 docker import [options] file|URL|- [REPOSITORY[:TAG]]
```

示例 
``` shell
docker import d:\docker-oracle.tar oracle11g:20210921**
```


以下内容为示例

1、选择要打包的镜像，执行打包命令 
docker save -o 打包镜像名称.tar（名称自定义） 镜像名称

docker save -o Cesium-1.tar tomcat

2、镜像打包完成后，会在当前目录下生成，使用ls命令查看

3、其他环境镜像导入该打包镜像 docker load -i 镜像名称

docker load -i Cesium-1.tar

4、容器打包镜像，打包完成后，使用ls命令查看
``` shell
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名： [TAG]
```

docker commit -a “xxx” -m “xxx” 容器名称或id 打包的镜像名称:标签

docker commit -a “sy” -m “三维html静态页面” cb045cd2afb6 cesium