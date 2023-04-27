title: docker学习
author: 陈龙
tags:
  - docker
categories:
  - 学习
date: 2023-04-27 14:47:00
---
## 安装环境说明

&emsp;&emsp; Docker官方建议在Ubuntu中安装，因为Docker是基于Ubuntu发布的，而且一般Docker出现的问题Ubuntu是最先更新或者打补丁的。在很多版本的CentOS中是不支持更新最新的一些补丁包的。
&emsp;&emsp; 由于我们学习的环境都使用的是CentOS，因此这里我们将Docker安装到CentOS上。注意：这里建议安装在CentOS7.x以上的版本，在CentOS6.x的版本中，安装前需要安装其他很多的环境而且Docker很多补丁不支持更新。

## 镜像加速  科学上网

网易的镜像地址：http://hub-mirror.c.163.com

新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon，在该配置文件中加入（没有该文件的话，请先建一个）：
```json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

## yum安装Docker

&emsp;&emsp; 从 2017 年 3 月开始 docker 在原来的基础上分为两个分支版本: Docker CE 和 Docker EE。
Docker CE 即社区免费版，Docker EE 即企业版，强调安全，但需付费使用。

&emsp;&emsp; Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

### 3.1 通过uname -r命令查看你当前的内核版本
``` shell
[root@localhost home]# uname -r
3.10.0-957.1.3.el7.x86_64
```

### 3.2 更新yum包
``` shell
[root@localhost home]# yum -y update
```

### 3.3 移除已安装的旧版本
``` shell
[root@localhost home]# yum remove docker \
                   docker-client \
                   docker-client-latest \
                   docker-common \
                   docker-latest \
                   docker-latest-logrotate \
                   docker-logrotate \
                   docker-selinux \
                   docker-engine-selinux \
                  docker-engine
```

### 3.4 安装必要的系统工具
```shell
[root@localhost home]# yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 3.5 添加软件源信息
```shell
[root@localhost home]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 3.6 更新yum缓存
```shell
[root@localhost home]# yum makecache fast
```

### 3.7 查看仓库中所有的docker版本，并选择特定版本安装
```shell
[root@localhost ~]# yum list docker-ce --showduplicates | sort -r
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
 * extras: mirrors.aliyun.com
 * epel: mirror.lzu.edu.cn
docker-ce.x86_64                3:18.09.0-3.el7                @docker-ce-stable
 * base: mirrors.aliyun.com
```

### 3.8 安装docker-ce
 由于repo中默认只开启stable仓库，故这里安装的是最新稳定版18.09.0
 ```shell
[root@localhost home]# yum -y install docker-ce
```

安装指定版本
```shell
[root@localhost home]# yum install <FQPN>  # 例如：sudo yum install docker-ce-18.09.0.ce
```

### 3.9 启动Docker后台服务
```shell
[root@localhost home]# systemctl start docker
或者
[root@localhost home]# service docker start
```

### 3.10 测试运行hello-world
由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行。
```shell
[root@localhost home]# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

### 3.11 查看Docker版本
```shell
[root@localhost home]# docker -v
Docker version 18.09.0, build 4d60db4
[root@localhost ~]# docker version
Client:
 Version:           18.09.0
 API version:       1.39
 Go version:        go1.10.4
 Git commit:        4d60db4
 Built:             Wed Nov  7 00:48:22 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.0
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.4
  Git commit:       4d60db4
  Built:            Wed Nov  7 00:19:08 2018
  OS/Arch:          linux/amd64
  Experimental:     false
```


## 4 脚本安装Docker

### 4.1 更新yum包
```shell
[root@localhost home]# yum update
```

### 4.2 下载并执行Docker安装脚本
执行这个脚本会添加 docker.repo 源并安装 Docker。
```shell
[root@localhost home]# curl -fsSL https://get.docker.com -o get-docker.sh
[root@localhost home]# sh get-docker.sh
```

### 4.3 启动Docker进程
```shell
[root@localhost home]# systemctl start docker
或者
[root@localhost home]# service docker start
```

### 4.4 测试运行hello-world
由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行。
```shell
[root@localhost home]# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### 4.5 查看Docker版本
```shell
[root@localhost home]# docker -v
Docker version 18.09.0, build 4d60db4
```

## 5 启动与停止Docker服务

systemctl命令是系统服务管理器指令，它是 service 和 chkconfig 两个命令组合。

### 5.1 启动
```shell
[root@localhost home]# systemctl start docker
或者
[root@localhost home]# service docker start
```

### 5.2 停止
```shell
[root@localhost home]# systemctl stop docker
或者
[root@localhost home]# service docker stop
```

### 5.3 重启
```shell
[root@localhost home]# systemctl restart docker
或者
[root@localhost home]# service restart stop
```

### 5.4 查看Docker状态
```shell
[root@localhost home]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2019-01-16 21:43:18 EST; 55min ago
     Docs: https://docs.docker.com
 Main PID: 7950 (dockerd)
    Tasks: 14
   Memory: 35.0M
   CGroup: /system.slice/docker.service
           └─7950 /usr/bin/dockerd -H unix://

Jan 16 21:43:17 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:17.834575284-05:00" level=info msg="Graph migration to content-addressability... seconds"
Jan 16 21:43:17 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:17.835788542-05:00" level=info msg="Loading containers: start."
Jan 16 21:43:17 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:17.956042119-05:00" level=info msg="Default bridge (docker0) is assigned with... address"
Jan 16 21:43:17 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:17.996025653-05:00" level=info msg="Loading containers: done."
Jan 16 21:43:18 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:18.023462688-05:00" level=info msg="Docker daemon" commit=4d60db4 graphdriver...n=18.09.0
Jan 16 21:43:18 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:18.023574853-05:00" level=info msg="Daemon has completed initialization"
Jan 16 21:43:18 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:18.024467667-05:00" level=warning msg="Could not register builder git source:...in $PATH"
Jan 16 21:43:18 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:18.030610952-05:00" level=info msg="API listen on /var/run/docker.sock"
Jan 16 21:43:18 localhost.localdomain systemd[1]: Started Docker Application Container Engine.
Jan 16 21:43:48 localhost.localdomain dockerd[7950]: time="2019-01-16T21:43:48.475953192-05:00" level=info msg="ignoring event" module=libcontainerd name...skDelete"
Hint: Some lines were ellipsized, use -l to show in full.
```

### 5.5 设置开机启动
```shell
[root@localhost home]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

### 5.6 关闭开机启动
```shell
[root@localhost home]# systemctl disable docker
Removed symlink /etc/systemd/system/multi-user.target.wants/docker.service.
```
### 5.7 查看Docker概要信息
```shell
[root@localhost home]# docker info
Containers: 5
 Running: 0
 Paused: 0
 Stopped: 5
Images: 2
Server Version: 18.09.0
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: c4446665cb9c30056f4998ed953e6d4ff22c7c39
runc version: 4fc53a81fb7c994640722ac585fa9ca548971871
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-957.1.3.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 4
Total Memory: 1.777GiB
Name: localhost.localdomain
ID: WYAF:ADYQ:U7R6:PYNW:RS3I:D4F7:BK4A:WTL2:UVZH:I3B4:YZ33:XBFV
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Registry Mirrors:
 http://hub-mirror.c.163.com/
Live Restore Enabled: false
Product License: Community Engine

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

## 6 删除Docker
```shell
[root@localhost home]# yum remove docker-ce
[root@localhost home]# rm -rf /var/lib/docker
```