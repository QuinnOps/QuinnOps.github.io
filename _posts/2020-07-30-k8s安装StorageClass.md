---
layout:     post
title:      k8s安装StorageClass
date:       2020-03-10
author:     Quinn
header-img: img/post-bg-debug.png
catalog: true
tags:
    - k8s
    - StorageClass
---



前提配置好nfs服务端

并确保客户端可以正常挂载

### 下载所需文件

下载所需文件，并进行内容调整

```
mkdir nfsvolume && cd nfsvolume
for file in class.yaml deployment.yaml rbac.yaml ; do wget https://raw.githubusercontent.com/kubernetes-incubator/external-storage/master/nfs-client/deploy/$file ; done
```

修改 deployment.yaml 中的两处 NFS 服务器 IP 和目录

```
...
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              value: 192.168.115.50
            - name: NFS_PATH
              value: /data/k8s
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.199.154
            path: /data/k8s
```

部署创建

```
kubectl create -f rbac.yaml
kubectl create -f class.yaml
kubectl create -f deployment.yaml
```

如果日志中看到“上有坏超级块”，请在集群内所有机器上安装nfs-utils并启动。

```
yum -y install nfs-utils
systemctl start nfs-utils
systemctl enable nfs-utils
rpcinfo -p
```

查看storageclass

```
kubectl get storageclass
NAME PROVISIONER RECLAIMPOLICY VOLUMEBINDINGMODE ALLOWVOLUMEEXPANSION AGE
managed-nfs-storage fuseim.pri/ifs Delete Immediate false 10m
```

### 标记一个默认的 StorageClass

操作命令格式如下

```
kubectl patch storageclass <your-class-name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

请注意，最多只能有一个 StorageClass 能够被标记为默认。

验证标记是否成功

```
kubectl get storageclass
NAME PROVISIONER RECLAIMPOLICY VOLUMEBINDINGMODE ALLOWVOLUMEEXPANSION AGE
managed-nfs-storage (default) fuseim.pri/ifs Delete Immediate false 12m
```

