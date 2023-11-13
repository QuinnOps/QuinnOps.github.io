---
layout:     post
title:      GlusterFS集群安装使用
date:       2023-10-20
author:     Quinn
header-img: img/post-bg-debug.png
catalog: true
tags:
    - linux
    - 分布式存储
    - glusterfs


---

创建lvm
fdisk -l
fdisk /dev/sdb  
pvcreate  /dev/sdb1
pvs
pvdisplay
vgcreate gfsvg /dev/sdb1
vgs
vgdisplay
lvcreate -l 100%free -n gfslv gfsvg  
lvs
lvdisplay
mkfs.ext4 /dev/gfsvg/gfslv 
blkid /dev/gfsvg/gfslv
mkdir /data
mount /dev/gfsvg//gfslv /data/
df -h
vim /etc/fstab 
umount /service 
mount -a

gfs集群 三台服务器192.168.148.145-147

一、安装集群，在每台服务器上做以下操作：

配置hosts解析

```
192.168.148.145  glusterfs01
192.168.148.146  glusterfs02
192.168.148.147  glusterfs03
```

关闭防火墙、selinux

```
systemctl stop firewalld.service
systemctl disable firewalld.service

setenforce 0
```

安装源

```
yum -y install centos-release-gluster
```

安装相关服务

```
yum -y install glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma
```

启动 glusterFS

```
systemctl start glusterd.service
```

设置自启动

```
systemctl enable glusterd.service
```

二、配置集群，在集群内任一节点操作（以01为例）：

查看集群的状态，当前信任池当中没有其他主机

```
gluster peer status 
```

添加集群信任池，将其他两台节点加入集群，本机不需要添加

```
gluster peer probe glusterfs02

gluster peer probe glusterfs03
```

查看集群信任池，此时显示有三个节点

```
gluster peer status
```



#### 检查集群节点存储池

```
gluster pool list
```



#### 创建存储卷

查看当前的卷

```
gluster volume list
```

创建卷（gfs有多种卷格式，见文末，这里创建的是分布式复制卷）

```
gluster volume create gfs_ai  replica 3 172.20.148.145:/service/gfs/ai/brick1 172.20.148.146:/service/gfs/ai/brick1 172.20.148.147:/service/gfs/ai/brick1 172.20.148.145:/service/gfs/ai/brick2 172.20.148.146:/service/gfs/ai/brick2 172.20.148.147:/service/gfs/ai/brick2 172.20.148.145:/service/gfs/ai/brick3 172.20.148.146:/service/gfs/ai/brick3 172.20.148.147:/service/gfs/ai/brick3
```

 查看卷的详细信息

```
gluster volume info
```



#### 启动刚刚创建的卷

```
gluster volume start gfs_ai  
```

客户端挂载

```
mount -t glusterfs 172.20.148.145:gfs_ai /service/test/
```

