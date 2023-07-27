---
title: 镜像操作
author: 陈龙
categories:
  - 技术
  - docker
tags:
  - docker
keywords:
  - docker
abbrlink: 1103407806
date: 2023-05-09 00:00:00
---
## 什么是docker镜像
&emsp;&emsp; 镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、库、环境变量和配置文件。
&emsp;&emsp; Docker镜像是由文件系统叠加而成（是一种文件的存储形式）。最底端是一个文件引导系统，即bootfs，这很像典型的Linux/Unix的引导文件系统。Docker用户几乎永远不会和引导系统有什么交互。实际上，当一个容器启动后，它将会被移动到内存中，而引导文件系统则会被卸载，以留出更多的内存供磁盘镜像使用。Docker容器启动是需要的一些文件，而这些文件就可以称为Docker镜像。

## UnionFS（联合文件系统，docker镜像构成原理）
一个docker镜像，由多层打包构成，从docker pull就可以看到：
```shell
[root@localhost /]# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
dc65f448a2e2: Pull complete 
346ffb2b67d7: Pull complete 
dea4ecac934f: Pull complete 
8ac92ddf84b3: Pull complete 
d8ef64070a18: Pull complete 
6577248b0d6e: Pull complete 
576c0a3a6af9: Pull complete 
6e0159bd18db: Pull complete 
8c831308dd9e: Extracting [=================================================> ]  11.14MB/11.34MB
c603174def53: Download complete

[root@localhost /]# docker rmi tomcat
Untagged: tomcat:latest
Untagged: tomcat@sha256:e895bcbfa20cf4f3f19ca11451dabc166fc8e827dfad9dd714ecaa8c065a3b18
Deleted: sha256:aeea3708743f98e54f26c49f3cefdabcb955f68c3cb097f2231e42918e678509
Deleted: sha256:be33a8bd23b7b4bbdb53f76b466d4d66c96eb74aef2177e9d7d6c5a3b7b70fc4
Deleted: sha256:c681be6da5ec4373f47c90e61f4ea395c63ac2339940a4832af6665fa8d39e95
Deleted: sha256:b5c6976d6644c3b38413f73ca6ccd72bb697b1916e5131bf533bfbb7ec35761f
Deleted: sha256:ac0fb599926db89be8716869d82aac3c6e62ea348773a69ad27c4224e8b86035
Deleted: sha256:b024e7dc232e8df80216d932be7a0e70861c98f5a18d28171b5d305b2e15fb72
Deleted: sha256:a426f9ec9cea304af7e78a6da88aeaecb8df1035bbc519018169da71a58a76a8
Deleted: sha256:79b74215e63df1e2e1b336d179f4cb31221aaf13eb7574bb31bc57265a407da1
Deleted: sha256:9af5ac7f79553c214feae23db6b73b26b676b4f5c49f6d9e5a3993eac01c2395
Deleted: sha256:9fe7a9e4f54169fec192350f6873eac00ec472d6341a2ea918bf5f877ed6941a
Deleted: sha256:ce8168f123378f7e04b085c9672717013d1d28b2aa726361bb132c1c64fe76ac
```

&emsp;&emsp; Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem）。UnionFS是docker镜像的基础，镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：
&emsp;&emsp; 一次同时加载多个文件系统，但是从外面看起来，只能看到一个系统文件，UnionFS会把隔层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。
## docker镜像加载原理
&emsp;&emsp; docker的镜像实际上是由一层一层的文件系统组成，这种层级的文件系统就是UnionFS。
bootfs（boot file system）主要包含bootloader和kernel，bootloader主要是引导加载kernel（Linux内核），linux刚启动时，会加载bootfs文件系统，在docker镜像的最底层就是bootfs（Linux内核），这一层与我们典型的linux/unix系统是一样的，包含boot加载器和内核。当boot加载完成后，整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。当docker容器启动完毕后，会自动卸载bootfs，只保留上层应用。

&emsp;&emsp; rootfs（root file system），在bootfs之上，包含的就是典型的linux系统中的/dev、/proc、/bin、/etc等标准目录和文件，rootfs就是各种不同的操作系统发行版，比如ubuntu、centos等。

&emsp;&emsp; 对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接使用宿主机的kernel（内核），容器只需要提供rootfs就行了，由此可见，对于不同的linux发行版，bootfs基本是一致的，rootfs会有差距，因此不同的发行版可以共用bootfs。
docker为什么快？就是因为docker共用宿主机的内核，只需根据不同的容器，启动各自的应用。

## docker镜像为什么要采用分层结构？
最大的一个好处就是：共享资源。
比如：
&emsp;&emsp; 有多个镜像都从相同的base镜像构建而来，那么宿主机只需要在磁盘上保存一份base镜像，同时，内存中也只需要加载一份base镜像，就可以为所有容器服务了，而且镜像的每一层都可以被共享。

## docker镜像的特点
&emsp;&emsp; 镜像是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部，这一层通常被称作容器层（可读可写），容器层之下都叫做镜像层。

## 搜索镜像
如果需要从网络中查找需要的镜像，可以通过以下命令搜索：
```shell
[root@localhost /]# docker search tomcat
NAME                          DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
tomcat                        Apache Tomcat is an open source implementati…   2648                [OK]                
tomee                         Apache TomEE is an all-Apache Java EE certif…   74                  [OK]                
dordoka/tomcat                Ubuntu 14.04, Oracle JDK 8 and Tomcat 8 base…   53                                      [OK]
bitnami/tomcat                Bitnami Tomcat Docker Image                     31                                      [OK]
```
字段内容：
- name
  - 镜像仓库名称
- description
  - 镜像仓库描述
- stars
  - 镜像收藏数
- official
  - 是否官方提供的镜像，该列标记为[OK]的镜像均由各软件的官方项目组创建和维护
- automated
  - 自动构建，标识该镜像由Docker Hub自动构建流程创建的

search参数说明：
- -s
  - 列出收藏数不小于指定值的镜像
- --no-trunc
  - 显示完整的镜像描述
- --automated
  - 只列出automated build类型的镜像

## 下载镜像
下载镜像：
```shell
[root@localhost ~]# docker pull redis
```
可以在镜像名称后面加 :版本号拉取指定版本镜像，例如下载centos:7，如果不加版本，默认拉latest版本
```shell
[root@localhost ~]# docker pull centos:7
```

## 列出镜像
```shell
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        2 weeks ago         1.84kB
ubuntu              15.10               9b9cb95443b5        2 years ago         137MB
```

- repository
  - 镜像所在的仓库名称，可以理解为镜像名称
- tag
  - 镜像标签，可以理解为版本，默认是latest，表示最新版本
- image id
  - 镜像id，唯一标识镜像的序列号
- created
  - 镜像的创建日期，不是获取该镜像的日期
- size
  - 镜像大小

这些镜像都存储在Docker宿主机的/var/lib/docker目录下。

镜像参数：
- -a：列出本地所有的镜像（含中间映像层）
- -q：只显示镜像id
- --digests：显示镜像的摘要信息
- --np-trunc：显示完整的镜像信息
## 删除镜像
### 9.1 删除指定镜像
docker rmi $IMAGE_ID
```shell
[root@localhost ~]# docker rmi 9b9cb95443b5
```
9b9cb95443b5是Ubuntu的IMAGE_ID。

### 9.2 删除所有镜像
docker rmi `docker images -q`，将` `中返回的结果交给docker rmi命令，-q参数表示查询IMAGE_ID
```shell
[root@localhost ~]# docker rmi `docker images -q`
或者
[root@localhost ~]# docker rmi $(docker images -q)
```
查询镜像的id：
```shell
[root@localhost ~]# docker images -q
fce289e99eb9
5d2989ac9711
```
### 9.3 删除镜像时报错
```shell
[root@localhost ~]# docker rmi 9b9cb95443b5
Error response from daemon: conflict: unable to delete 9b9cb95443b5 (must be forced) - image is being used by stopped container 9377ba1f79f8
```
解决办法：
- 先查询容器
```shell
docker ps -a
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS               NAMES
329013443c1f        hello-world         "/hello"                 6 hours ago         Exited (0) 6 hours ago                       hopeful_bell
9377ba1f79f8        ubuntu:15.10        "/bin/echo 'Hello wo…"   2 days ago          Exited (0) 2 days ago                        friendly_leakey
```

将要删除镜像的容器删除
```shell
[root@localhost ~]# docker rm 9377ba1f79f8
9377ba1f79f8
```

再根据IMAGE_ID删除镜像即可

或者强制删除镜像（推荐）
```shell
[root@localhost ~]# docker rmi -f hello-world
```

## 提交镜像
commit是合并了save、load、export、import这几个特性的一个综合性的命令，它主要做了：
- 将container当前的读写层保存下来，保存成一个新层
- 和镜像的历史层一起合并成一个新的镜像

如果原本的镜像有3层，commit之后就会有4层，最新的一层为从镜像创建容器并运行，到commit之间对文件系统的修改

docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名[:标签名]
```shell
[root@localhost ~]# docker commit -m="rediscommit" -a="wyc" c30d310ce625 redis-commit:v1.0
sha256:9b99689606a0b5f86497e4e8853dd1b732459b69231ab94cc0921345e10af973
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redis-commit        v1.0                9b99689606a0        3 seconds ago       98.2MB
```

提交后的镜像可以用来创建新的容器