---
title: "入门教程"
description: "夜莺（ Nightingale ）入门视频教程"
lead: ""
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "usage"
weight: 800
toc: true
---

- [视频讲解-夜莺页面功能介绍](https://mp.weixin.qq.com/s/6fZBGq0o1o4wVRlKdKSfrA)
- [视频讲解-告警自愈脚本的使用](https://mp.weixin.qq.com/s/4xlbWjclbSyCn91HpVRT9g)
- [视频讲解-监控对象的管理功能](https://mp.weixin.qq.com/s/Tg4Q2lhdTiDSu-jCm8vVwQ)
- [视频讲解-如何接入多个时序存储](https://mp.weixin.qq.com/s/ThFjha8ocjdz3sqI1Pjwjg)
- [视频讲解-夜莺的配置文件](https://mp.weixin.qq.com/s/t0bTahPYoLsi_bGvNFspkw)

除了这些视频教程，使用手册这一章，还会提供更多小节来介绍一些大家常问的问题，比如如何监控交换机，如何监控应用等。从总体问题比例来看，夜莺服务端问题相对较少，大家摸索一下，很快可以掌握，疑问比较多的，是对各种目标的监控，比如如何监控 MySQL，如何监控 Redis 等，针对这部分问题，大家应该去查看采集器的文档，比如 Telegraf 每个采集器下面都有 [README介绍](https://github.com/influxdata/telegraf/tree/master/plugins/inputs)， 通过这些 README，理论上就可以知道如何使用各个采集器。Categraf 的采集器目录 [在这里](https://github.com/flashcatcloud/categraf/tree/main/inputs)， 未来我们也会把所有采集器补充完整的 README，以及告警规则和监控大盘JSON，大家导入直接就可以使用，当然，路漫漫其修远兮，一步一步来吧。