---
layout:     post
title:      kafka针对单个topic设置数据保留时间
date:       2021-04-21
author:     Quinn
header-img: img/post-bg-debug.png
catalog: true
tags:
    - kafka


---

### 全局设置

修改server.properties

```markup
log.retention.hours=72
log.cleanup.policy=delete
```

### 单独对某一个topic设置保留时间

设置成保留4个小时，单位是毫秒

```markup
./bin/kafka-configs.sh --bootstrap-server 192.168.10.10:9092 --alter --entity-name rundata --entity-type topics --add-config retention.ms=14400000
```

查看设置

```markup
./bin/kafka-configs.sh --bootstrap-server 192.168.10.10:9092 --describe --entity-name rundata --entity-type topics
```
