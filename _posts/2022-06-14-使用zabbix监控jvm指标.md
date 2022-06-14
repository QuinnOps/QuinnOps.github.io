---
layout:     post
title:      使用zabbix监控jvm指标
date:       2022-06-14
author:     Quinn
header-img: img/post-bg-debug.png
catalog: true
tags:
    - jvm
    - java
    - zabbix


---

安装、配置  zabbix-java-gateway

```shell
[root@zabbix ~]# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
[root@zabbix ~]# yum -y install zabbix-java-gateway
[root@zabbix ~]# vi /etc/zabbix/zabbix_java_gateway.conf 
LISTEN_IP="0.0.0.0" 监听所有地址
LISTEN_PORT=10052	监听端口
START_POLLERS=5		启动线程
TIMEOUT=3			超时时长（秒）
[root@zabbix ~]# systemctl start zabbix-java-gateway
[root@zabbix ~]# systemctl enable zabbix-java-gateway
```

配置服务启动参数（实测配置端口、认证和ssl即可）

```shell
//启用远程监控JMX
-Dcom.sun.management.jmxremote
//jmx启用远程端口,Zabbix添加时必须一致
-Dcom.sun.management.jmxremote.port=12345
//不开启用户密码认证
-Dcom.sun.management.jmxremote.authenticate=false
//不启用ssl加密传输
-Dcom.sun.management.jmxremote.ssl=false
//运行tomcat主机的IP地址
-Djava.rmi.server.hostname=10.0.0.7"
//启用密码验证(jmxremote.password为密码文件，第一列是用户名，第二列是密码，文件权限600)
-Dcom.sun.management.jmxremote.password.file=jmxremote.password
```

修改zabbix server或proxy端配置，需重启

ip 和端口根据实际填写，本例server 和java gateway 安装在同一台主机上，所以填写127.0.0.1
StartJavaPollers 应小于等于zabbix_java_gateway.conf 中START_POLLERS的值。参考官方文档
https://www.zabbix.com/documentation/4.0/zh/manual/concepts/java?s[]=java&s[]=gateway

```shell
JavaGateway=127.0.0.1
JavaGatewayPort=10052
StartJavaPollers=5
```

通过zabbix web 配置主机，如图，添加jmx配置和模板

![](https://github.com/QuinnOps/QuinnOps.github.io/blob/master/img/zabbix-jvm.jpg)

![](https://github.com/QuinnOps/QuinnOps.github.io/blob/master/img/zabbix-jvm-template.jpg)

