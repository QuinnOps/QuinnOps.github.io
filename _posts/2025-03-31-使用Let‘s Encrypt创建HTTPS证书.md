---
layout:     post
title:      使用Let‘s Encrypt创建免费的TLS证书
date:       2025-03-31
author:     Quinn
header-img: img/post-bg-debug.png
catalog: true
tags:
    - nginx
    - SSL
    - TLS
    - Let‘s Encrypt
    - Certbot


---

**Let's Encrypt** 是一个免费、自动化且开放的**证书颁发机构（CA）**，致力于推动全球网站的 **HTTPS 加密化**。它由 **Internet Security Research Group (ISRG)** 于 2016 年推出，旨在降低 SSL/TLS 证书的获取门槛，让所有网站都能轻松实现安全加密连接。

**Certbot** 是由 **Electronic Frontier Foundation (EFF)** 开发的一款开源工具，用于自动化获取、部署和续期 **Let's Encrypt** 的 SSL/TLS 证书。它是目前最流行的 **ACME 协议客户端**，支持多种操作系统和 Web 服务器，帮助用户轻松实现网站 HTTPS 加密。

本篇文章将介绍如何使用Certbot客户端来创建Let‘s Encrypt证书。

### 一. 安装Certbot

Certbot官网提供了安装方式，选择好安装证书的软件和操作系统即可输出对应的安装使用步骤。

如果是Linux系统，这里推荐使用snap安装Certbot，不推荐使用pip安装的方式。

本人使用pip安装Certbot遇到几个问题:

> Certbot即将不支持python3.6及以下版本

> Centos7安装python3版本可能会和自带python2版本冲突

> python虚拟环境运行Certbot urllib3依赖openssl1.1.1+版本（Centos7系统自带openssl1.0.2），升级openssl也会带来版本管理和重新指定openssl编译python等问题

而使用snap安装则可以让 certbot 使用独立的 Python 环境。以Centos7上运行NGINX为例，安装步骤：

#### 1、卸载旧版 certbot

```
#sudo yum remove certbot -y
```

 #### 2、安装 snapd

```
#sudo yum install -y snapd
#sudo systemctl enable --now snapd.socket
#sudo ln -s /var/lib/snapd/snap /snap
```


 #### 3、通过 snap 安装 certbot

```
#sudo snap install --classic certbot
#sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 二. 使用Certbot--生成证书及自动续期

#### 1、生成证书
运行 certbot，会提示输入邮箱地址，以及注册到acme等，选择需要生成证书的域名后，Certbot会将生成的证书直接配置到nginx对应的配置文件中。
生成的证书路径：/etc/letsencrypt/live/，有效期为90天。

```
#sudo certbot --nginx

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): xxx@xxx.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.5-February-24-2025.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n
Account registered.

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: blog.quinnops.eu.org
2: xxx.quinnops.eu.org
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 2
Requesting a certificate for xxx.quinnops.eu.org

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/xxx.quinnops.eu.org/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/xxx.quinnops.eu.org/privkey.pem
This certificate expires on 2025-06-26.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for xxx.quinnops.eu.org to /etc/nginx/conf.d/xxx.quinnops.eu.org.conf
Congratulations! You have successfully enabled HTTPS on https://xxx.quinnops.eu.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

#### 2、自动续期

Certbot 默认会创建自动续期任务，手动测试续期：

```
#sudo certbot renew --dry-run

Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/xxx.quinnops.eu.org.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Account registered.
Simulating renewal of an existing certificate for xxx.quinnops.eu.org

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded: 
  /etc/letsencrypt/live/xxx.quinnops.eu.org/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

#### 3、查看自动续期定时任务

```
#sudo systemctl list-timers | grep certbot

Fri 2025-03-28 20:19:00 GMT  10h left n/a                          n/a    snap.certbot.renew.timer     snap.certbot.renew.service
```

#### 4、手动调整续期时间

默认情况下，Certbot 会每天检查证书是否接近过期（30天内到期时自动续期）。如果想修改检查频率，可以编辑 systemd 定时器：

```
#sudo systemctl edit certbot.timer
```

添加以下内容（例如调整为每周检查一次）：

```
[Timer]
OnCalendar=weekly
RandomizedDelaySec=1h
```

然后重新加载 systemd：

```
#sudo systemctl daemon-reload
```

#### 5、强制立即续期

```
#sudo certbot renew --force-renewal
```

#### 6、验证证书有效期

```
#sudo certbot certificates

Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: xxx.quinnops.eu.org
    Serial Number: 5e89f5b0445efb48ac5f2c05b79xxxxxxx
    Key Type: ECDSA
    Domains: xxx.quinnops.eu.org
    Expiry Date: 2025-06-26 08:17:54+00:00 (VALID: 86 days)
    Certificate Path: /etc/letsencrypt/live/xxx.quinnops.eu.org/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/xxx.quinnops.eu.org/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

#### 7、续期后自动重启 Nginx

在 `/etc/letsencrypt/renewal-hooks/post` 下创建脚本并赋予执行权限：

```
#sudo mkdir -p /etc/letsencrypt/renewal-hooks/post
#sudo vim /etc/letsencrypt/renewal-hooks/post/restart-nginx.sh
#sudo chmod +x /etc/letsencrypt/renewal-hooks/post/restart-nginx.sh
```

内容：

```
#!/bin/bash
systemctl restart nginx
```

