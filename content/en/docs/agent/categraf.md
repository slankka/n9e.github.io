---
title: "Categraf"
description: "Categraf是一款all-in-one的采集器，是夜莺主推的一款监控客户端"
lead: ""
date: 2022-05-12T13:26:54+01:00
lastmod: 2022-05-12T13:26:54+01:00
draft: false
images: []
menu:
  docs:
    parent: "agent"
weight: 700
toc: true
---


Categraf 是一款 all-in-one 的采集器，由 [快猫团队](https://flashcat.cloud) 开源，代码托管在两个地方：

- gitlink: [https://www.gitlink.org.cn/flashcat/categraf](https://www.gitlink.org.cn/flashcat/categraf)
- github: [https://github.com/flashcatcloud/categraf](https://github.com/flashcatcloud/categraf)

Categraf 不但可以采集 OS、MySQL、Redis、Oracle 等常见的监控对象，也准备提供日志采集能力和 trace 接收能力，这是夜莺主推的采集器，相关信息请查阅项目 [README](https://www.gitlink.org.cn/flashcat/categraf/tree/main/README.md)

Categraf 采集到数据之后，通过 remote write 协议推给远端存储，Nightingale 恰恰提供了 remote write 协议的数据接收接口，所以二者可以整合在一起，重点是配置 Categraf 的 `conf/config.toml` 中的 writer 部分，其中 url 部分配置为 n9e-server 的 remote write 接口：

```toml
[writer_opt]
# default: 2000
batch = 2000
# channel(as queue) size
chan_size = 10000

[[writers]]
url = "http://N9E-SERVER:19000/prometheus/v1/write"

# Basic auth username
basic_auth_user = ""

# Basic auth password
basic_auth_pass = ""

# timeout settings, unit: ms
timeout = 5000
dial_timeout = 2500
max_idle_conns_per_host = 100
```