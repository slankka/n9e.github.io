---
title: "SNMP"
description: "夜莺（ Nightingale ）通过 SNMP 监控网络设备"
lead: ""
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "usage"
weight: 810
toc: true
---

监控网络设备，主要是通过 SNMP 协议，Categraf、Telegraf、Datadog-Agent、snmp_exporter 都提供了这个能力。

## Categraf

Categraf 提供了一个网络设备的采集插件：switch_legacy，在 `conf/input.switch_legacy` 下可以看到配置文件，最核心就是配置交换机的 IP 以及认证信息，switch_legacy 当前只支持 v2 协议，所以认证信息就是 community 字段。其他配置都一目了然，这里就不赘述了。

这个插件是把之前 Open-Falcon 社区的 [swcollector](https://github.com/gaochao1/swcollector) 直接拿过来了，感谢 `冯骐` 大佬持续在维护这个开源项目。

## Telegraf

Telegraf 内置支持 SNMP 的采集，本节给一个入门例子，让大家快速上手，更多具体知识可以参考[这里](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/snmp)。在 telegraf.conf 中搜索 inputs.snmp，即可找到对应的配置，例子如下：

```toml
[[inputs.snmp]]
agents = ["udp://172.25.79.194:161"]
timeout = "5s"
version = 3
agent_host_tag = "ident"
retries = 1
sec_name = "managev3user"
auth_protocol = "SHA"
auth_password = "example.Demo.c0m"

[[inputs.snmp.field]]
oid = "RFC1213-MIB::sysUpTime.0"
name = "uptime"

[[inputs.snmp.field]]
oid = "RFC1213-MIB::sysName.0"
name = "source"
is_tag = true

[[inputs.snmp.table]]
oid = "IF-MIB::ifTable"
name = "interface"
inherit_tags = ["source"]

[[inputs.snmp.table.field]]
oid = "IF-MIB::ifDescr"
name = "ifDescr"
is_tag = true
```

上面非常关键的部分是：`agent_host_tag = "ident"`，因为夜莺对 ident 这个标签会特殊对待处理，把携有这个标签的数据当做隶属某个监控对象的数据，机器和网络设备都是典型的期望作为监控对象来管理的，所以 SNMP 的采集中，我们把网络设备的 IP 放到 ident 这个标签里带上去。

另外这个采集规则是 v3 的校验方法，不同的公司可能配置的校验方式不同，请各位参照 telegraf.conf 中那些 SNMP 相关的注释仔细核对，如果是 v2 会简单很多，把上例中的如下部分：

```toml
version = 3
sec_name = "managev3user"
auth_protocol = "SHA"
auth_password = "example.Demo.c0m"
```

换成：

```toml
version = 2
community = "public"
```

即可，当然了，community 要改成你们自己的，这里写的 “public” 只是举个例子。

`inputs.snmp.field` 相关的那些配置，可以采集到各个网口的监控指标，更多的使用方式请参考[官网](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/snmp)

另外，snmp的采集，建议大家部署单独的 Telegraf 来做，因为和机器、中间件等的采集频率可能不同，比如边缘交换机，我们 5min 采集一次就够了，如果按照默认的配置可是 10s 采集一次，实在是太频繁了，可能会给一些老式交换机造成比较大的压力，采集频率在 telegraf.conf 的最上面 `[agent]` 部分，边缘交换机建议配置为：

```toml
[agent]
interval = "300s"
flush_interval = "300s"
```

核心交换机可以配置的频繁一些，比如 60s 或者 120s，请各位网络工程师朋友自行斟酌。
