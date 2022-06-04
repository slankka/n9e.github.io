---
title: "数据推送"
description: "如何把自定义监控数据推送给夜莺Nightingale"
lead: ""
date: 2020-11-12T13:26:54+01:00
lastmod: 2020-11-12T13:26:54+01:00
draft: false
images: []
menu:
  docs:
    parent: "api"
weight: 910
toc: true
---

在 采集器 章节可以看出，夜莺支持多种数据接收的接口（由 n9e-server 实现，推送数据就是推给 n9e-server 的 19000 端口），包括 OpenTSDB、Open-Falcon、RemoteWrite、Datadog 等协议。这节我们以 OpenTSDB 的数据接收接口举例。

OpenTSDB 的数据接收接口的 Url Path 是 `/opentsdb/put` ，POST 方法，监控数据做成 JSON 放到 HTTP Request Body 中，举例：

```json
[
	{
		"metric": "cpu_usage_idle",
		"timestamp": 1637732157,
		"tags": {
			"cpu": "cpu-total",
			"ident": "c3-ceph01.bj"
		},
		"value": 30.5
	},
	{
		"metric": "cpu_usage_util",
		"timestamp": 1637732157,
		"tags": {
			"cpu": "cpu-total",
			"ident": "c3-ceph01.bj"
		},
		"value": 69.5
	}
]
```

显然，JSON 最外层是个数组，如果只上报一条监控数据，也可以不要外面的中括号，直接把对象结构上报：

```json
{
	"metric": "cpu_usage_idle",
	"timestamp": 1637732157,
	"tags": {
		"cpu": "cpu-total",
		"ident": "c3-ceph01.bj"
	},
	"value": 30.5
}
```

服务端会看第一个字符是否是`[`，来判断上报的是数组，还是单个对象，自动做相应的 Decode。如果觉得上报的内容太过占用带宽，也可以做 gzip 压缩，此时上报的数据，要带有`Content-Encoding: gzip`的 Header。

{{< alert icon="💡" text="注意 ident 这个标签，ident 是 identity 的缩写，表示设备的唯一标识，如果标签中有 ident 标签，n9e-server 就认为这个监控数据是来自某个机器的，会自动获取 ident 的 value，注册到监控对象的列表里" />}}
