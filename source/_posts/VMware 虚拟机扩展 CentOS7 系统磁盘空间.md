---
title: VMware 虚拟机扩展 CentOS7 系统磁盘空间
author: 陈龙
tags:
  - linux
  - vmware
categories:
  - linux
  - 技术
  - vsphere
keywords:
  - 扩容
  - linux
date: 2023-07-13 09:48
---

# 1.虚拟机扩展磁盘容量

关闭Vmware的centos7系统，才能在VMWare菜单中设置需要增加到的磁盘大小。同时要保证该系统没有快照。如果有快照的话，就无法直接扩展（这个功能无法使用），需要通过增加磁盘的方式扩展。  
![在这里插入图片描述](1f0934333a2e2c88e861f56c80655ad0_MD5.png)  

这里填写最终的磁盘大小，点击扩展。

![在这里插入图片描述](6f9719b60fa37ae65955e772e816b93e_MD5.png) 

这里的扩展只是增加了操作系统的磁盘空间，并没有与系统内部的文件目录挂载，所以，磁盘占有量还是不会变化，下一步就是要把扩展的容量挂载到文件目录上去。

如果系统中有快照，需要增加磁盘，相当于类似我们服务器没有存储空间的时候增加一块磁盘的原理。
# 2.扩展系统磁盘容量

查看待扩展的磁盘总空间。  

``` shell
lsblk 
```
![在这里插入图片描述](20656a7627163f016b8e4ed261d64096_MD5.png)
对新增加的硬盘进行分区

``` shell
fdisk /dev/sda
```
![在这里插入图片描述](21607dc5aad82c545482c0d90a1d54c1_MD5.png) 
分区的位置除了默认回车的方式，还可以自己根据区间值设置，最好是从头开始，这样磁盘不会存在中间有个空档。（36456448-125829119）根据系统提示的这个数字设置。

w 是写入这个分区表。
修改磁盘信息  
fdisk -l 发现sda3 的Id 是83 我们要将它改成8e跟sda2是一样的 将system 类型改成Linux LVM  
![在这里插入图片描述](5dcb624516989f9d86969256f638b8e5_MD5.png)

执行命令：fdisk /dev/sda  
![在这里插入图片描述](4225ff6944406c61086aa9522c0e0c4b_MD5.png)

```shell
fdisk -l 再查看一下是否改成8e 和Linux LVM
```
![在这里插入图片描述](96a443f59bf1fece986267d0f032f9e0_MD5.png)

重启系统 ：
``` shell
shutdown -r now
```

# 3.对新增加的硬盘格式化

将文件格式改成ext4的，执行命令：
``` shell
mkfs.ext4 /dev/sda3
```
![在这里插入图片描述](0008f19e09bf9338289a8bafdbe47fdf_MD5.png)

# 4.添加新LVM到已有的LVM组，实现扩容

创建sda3 ： 
``` shell
pvcreate /dev/sda3
```
![在这里插入图片描述](cf1db9f1ebb586eea5fa9f2493a65906_MD5.png) 

用命令：``pvdisplay`` 进行查看是否创建成功  
![在这里插入图片描述](8adbaf3ecdf0258b63cf29aedccfc454_MD5.png)

这里操作要根据上图中VG Name来定义用vgextends谁，我这里是centos那么我就用centos执行下面命令：
```shell
vgextend centos /dev/sda3
```
![在这里插入图片描述](2f8b7a0d67324c09aaeff73b55044465_MD5.png)

用命令：
``` shell 
pvdisplay
``` 
进行查看修改成功没有  
![在这里插入图片描述](30b17d796b5dd0c29639432ffba9e9b0_MD5.png)

执行命令：
``` shell
lvextend -L +42G /dev/mapper/centos-root 
```
进行扩容，+42G数字，自己根据情况定义  
![在这里插入图片描述](66e51d7ce3287c79da43c0ba845b92eb_MD5.png)

如果发现报错，可能是设置的扩如大小超出了本身拥有的大小，这个与我们查询到的值会有几MB的误差。我系统显示的是<42.6Gib,所以我使用了42G，大家也可以用过计算得到：

执行 ``pvdisplay`` 可以看到 /dev/sda3 可用的 PE 总数量是 10909，而每个 PE 大小是 4.00MiB，所以其实这个卷组实际的可用空间其实不是显示的值，

（10909 * 4 ）/ 1024 =42.61328125G 而是42.61328125G。

执行命令：`lvs` 进行查看是否成功，很显然，我的空间扩展成功了
查看磁盘空间采用的文件系统 df -T
![在这里插入图片描述](be2cd4e5d8acda1535aa2822abd86494_MD5.png)

执行命令：
```shell
xfs_growfs /dev/mapper/centos-root
```
![在这里插入图片描述](a496634f1d21791783b5e2635f558ddf_MD5.png)

```shell
df -h 查看
```  
![在这里插入图片描述](372aebb8d9b4763bed8d8ff682801ce5_MD5.png)