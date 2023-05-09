---
title: 镜像导出
author: 陈龙
categories:
  - 技术
  - docker
tags:
  - docker
keywords:
  - docker
abbrlink: 493888480
date: 2023-05-09 00:00:00
---
## save/load
### 1.1 save
docker save可以导出一个或多个镜像
```shell
docker save -o images.tar redis:9.6 nginx:3.4
```

打包之后的images.tar包含redis:9.6和nginx:3.4这两个镜像。

&emsp;&emsp; docker save的应用场景是，如果你的应用是使用docker-compose.yml编排的多个镜像组合，但你要部署的客户服务器并不能连外网。这时，你可以使用docker save将用到的镜像打个包，然后拷贝到客户服务器上使用docker load载入。

- **使用save导出时，保存镜像的history历史记录**

### 1.2 load
将打包后的镜像载入进来使用docker load，例如：
```shell
docker load -i images.tar
```

上述命令将会把redis:9.6和nginx:3.4载入进来，如果本地镜像库已经存在这两个镜像，将会被覆盖。

### 1.3 查看历史操作记录

使用load导入save导出的镜像时，不可以指定镜像的别名和标签，只能还原为原来版本的镜像
```shell
# 查看镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
tomcat              latest              4e7840b49fad        17 hours ago         529MB
# 将已存在镜像导出为指定镜像
[root@localhost ~]# docker save -o /tomcat-save.tar tomcat
# 加载指定镜像，但是不能指定导入后的镜像名和版本号（标签）
[root@localhost ~]# docker load -i /tomcat-save.tar 
Loaded image: tomcat:latest
# 查看导入后的镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat              latest              4e7840b49fad        17 hours ago        529MB
# 查看镜像的历史操作记录
[root@localhost ~]# docker history tomcat
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
4e7840b49fad        17 hours ago        /bin/sh -c #(nop)  CMD ["catalina.sh" "run"]    0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  EXPOSE 8080                  0B                  
<missing>           17 hours ago        /bin/sh -c set -e  && nativeLines="$(catalin…   0B                  
<missing>           17 hours ago        /bin/sh -c set -eux;   savedAptMark="$(apt-m…   19MB                
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_SHA512=bb9207d…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_VERSION=8.5.51    0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_MAJOR=8           0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV GPG_KEYS=05AB33110949…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV LD_LIBRARY_PATH=/usr/…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_NATIVE_LIBDIR=…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop) WORKDIR /usr/local/tomcat     0B                  
<missing>           17 hours ago        /bin/sh -c mkdir -p "$CATALINA_HOME"            0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV PATH=/usr/local/tomca…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV CATALINA_HOME=/usr/lo…   0B                  
<missing>           29 hours ago        /bin/sh -c set -eux;   dpkgArch="$(dpkg --pr…   205MB               
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_URL_VERSION=8u24…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_BASE_URL=https:/…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_VERSION=8u242       0B                  
<missing>           29 hours ago        /bin/sh -c { echo '#/bin/sh'; echo 'echo "$J…   27B                 
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV PATH=/usr/local/openj…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/local/…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           29 hours ago        /bin/sh -c set -eux;  apt-get update;  apt-g…   11.1MB              
<missing>           2 days ago          /bin/sh -c apt-get update && apt-get install…   145MB               
<missing>           2 days ago          /bin/sh -c set -ex;  if ! command -v gpg > /…   17.5MB              
<missing>           2 days ago          /bin/sh -c apt-get update && apt-get install…   16.5MB              
<missing>           2 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           2 days ago          /bin/sh -c #(nop) ADD file:e05e45c33042db4ec…   114MB
```

## export/import
### 2.1 export
docker export可以将容器打包为镜像
```shell
docker export -o redis-export.tar redis_id
```

- **使用export导出时，不保存镜像的history历史记录**

### 2.2 import
载入由容器打包的镜像，注意，即使是由容器打包成镜像，载入后，依然作为镜像存在，需要重新创建容器
```shell
docker export -o redis-export.tar redis_id
```

- redis-new：指定repository
- latest：指定镜像版本
### 2.3 操作历史记录
使用import导入export导出容器时，可以指定镜像的名称和标签
```shell
# 查看容器
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c1d8194e2a88        tomcat              "catalina.sh run"   9 minutes ago       Up 9 minutes        8080/tcp            zealous_shtern
# 在容器中创建文件，修改容器的操作记录
[root@localhost ~]# docker exec c1d8194e2a88 touch /123.txt
# 将容器导出为镜像
[root@localhost ~]# docker export -o /tomcat-export.tar c1d8194e2a88
# 导入修改后的镜像，同时指定导入后的镜像名称和版本号（标签）
[root@localhost ~]# docker import /tomcat-export.tar tomcat-export:1.0
sha256:1c307594a95e825652f4e68582684168c2fbf2a119024bdcb3c1fec2ccba08a3
# 查看导入后的镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat-export       1.0                 1c307594a95e        3 seconds ago       521MB
tomcat              latest              4e7840b49fad        17 hours ago        529MB
# 查看操作历史，可以看到使用export导出的镜像，并未保存镜像历史
[root@localhost ~]# docker history 1c307594a95e
IMAGE               CREATED             CREATED BY          SIZE                COMMENT
1c307594a95e        11 seconds ago                          521MB               Imported from -
```

## commit
### 3.1 commit
docker commit可以将容器打包为镜像，并且保留镜像的操作历史记录
```shell
[root@localhost ~]# docker commit -m="tomcat-commit" -a="wyc" c1d8194e2a88 tomcat-commit:2.0
sha256:71adaad17fde2b724ac99889c490d0598bcdfdff0b65330242d5a7d83dc21629
```

- **打包后的镜像，保留镜像的历史操作记录**
### 3.2 操作历史记录
```shell
# 查看容器
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c1d8194e2a88        tomcat              "catalina.sh run"   22 minutes ago      Up 22 minutes       8080/tcp            zealous_shtern
# 将容器提交为镜像
[root@localhost ~]# docker commit -m="tomcat-commit" -a="wyc" c1d8194e2a88 tomcat-commit:2.0
sha256:71adaad17fde2b724ac99889c490d0598bcdfdff0b65330242d5a7d83dc21629
# 查看提交后的镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat-commit       2.0                 71adaad17fde        11 seconds ago      529MB
tomcat              latest              4e7840b49fad        17 hours ago        529MB
# 查看镜像的历史记录，可以看到保留了原有镜像的历史记录
[root@localhost ~]# docker history 71adaad17fde
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
71adaad17fde        40 seconds ago      catalina.sh run                                 37.2kB              tomcat-commit
4e7840b49fad        17 hours ago        /bin/sh -c #(nop)  CMD ["catalina.sh" "run"]    0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  EXPOSE 8080                  0B                  
<missing>           17 hours ago        /bin/sh -c set -e  && nativeLines="$(catalin…   0B                  
<missing>           17 hours ago        /bin/sh -c set -eux;   savedAptMark="$(apt-m…   19MB                
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_SHA512=bb9207d…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_VERSION=8.5.51    0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_MAJOR=8           0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV GPG_KEYS=05AB33110949…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV LD_LIBRARY_PATH=/usr/…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV TOMCAT_NATIVE_LIBDIR=…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop) WORKDIR /usr/local/tomcat     0B                  
<missing>           17 hours ago        /bin/sh -c mkdir -p "$CATALINA_HOME"            0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV PATH=/usr/local/tomca…   0B                  
<missing>           17 hours ago        /bin/sh -c #(nop)  ENV CATALINA_HOME=/usr/lo…   0B                  
<missing>           29 hours ago        /bin/sh -c set -eux;   dpkgArch="$(dpkg --pr…   205MB               
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_URL_VERSION=8u24…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_BASE_URL=https:/…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_VERSION=8u242       0B                  
<missing>           29 hours ago        /bin/sh -c { echo '#/bin/sh'; echo 'echo "$J…   27B                 
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV PATH=/usr/local/openj…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/local/…   0B                  
<missing>           29 hours ago        /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           29 hours ago        /bin/sh -c set -eux;  apt-get update;  apt-g…   11.1MB              
<missing>           2 days ago          /bin/sh -c apt-get update && apt-get install…   145MB               
<missing>           2 days ago          /bin/sh -c set -ex;  if ! command -v gpg > /…   17.5MB              
<missing>           2 days ago          /bin/sh -c apt-get update && apt-get install…   16.5MB              
<missing>           2 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           2 days ago          /bin/sh -c #(nop) ADD file:e05e45c33042db4ec…   114MB
```

## save、export和commit的区别
- save保存的是镜像（image），export保存的是容器（container）、commit保存的是容器（container）
- load用来载入镜像包，import用来载入容器包，但两者都会恢复为镜像
- load不能对载入的镜像重命名，import可以为镜像指定新名称和标签（版本）
- export不保存容器的历史记录，commit可以保存容器的历史记录













