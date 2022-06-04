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

页面上的功能挨个描述的话篇幅太长了，录制了一套小视频教程供大家参考，不过这套视频教程比较浅显，基于5.0的版本介绍的，相对较老，夜莺一直在快速迭代，增加了很多新特性，有很多特性没有在视频教程中体现，请大家见谅，后面会再录制一套更详尽的，基于新版本的讲解视频。

- [01-使用Docker Compose一行命令安装夜莺v5](https://mp.weixin.qq.com/s/V_e14pBb2s14CGM1fl018A)
- [02-快速在生产环境部署启动单机版夜莺v5](https://mp.weixin.qq.com/s/r60zCkPWV3Lpl2QdzVm_gg)
- [03-讲解夜莺v5人员组织相关功能](https://mp.weixin.qq.com/s/Sdigb4r7bNFLxGfHhiYlqw)
- [04-讲解夜莺v5监控看图相关功能](https://mp.weixin.qq.com/s/zg0ONha7PLoY5KrITNk6IA)
- [05-讲解夜莺v5告警规则的使用](https://mp.weixin.qq.com/s/D2mNS1rRyw1XGFVJFrwMYw)
- [06-讲解夜莺v5告警屏蔽规则的使用](https://mp.weixin.qq.com/s/4lHrgGGsNEMBsl1mr0uS3A)
- [07-讲解夜莺告警订阅规则的使用](https://mp.weixin.qq.com/s/jKuu345kj-eDVMpCsmNUGA)
- [08-讲解夜莺v5活跃告警和历史告警](https://mp.weixin.qq.com/s/W2gycPgBWDSv73Xqbl_d6Q)
- [09-讲解夜莺v5告警自愈脚本的使用](https://mp.weixin.qq.com/s/4xlbWjclbSyCn91HpVRT9g)
- [10-讲解夜莺监控对象的管理功能](https://mp.weixin.qq.com/s/Tg4Q2lhdTiDSu-jCm8vVwQ)
- [11-夜莺v5如何接入多个时序存储](https://mp.weixin.qq.com/s/ThFjha8ocjdz3sqI1Pjwjg)
- [12-讲解夜莺v5配置文件](https://mp.weixin.qq.com/s/t0bTahPYoLsi_bGvNFspkw)

除了这些视频教程，使用手册这一章，还会提供更多小节来介绍一些大家常问的问题，比如如何监控交换机，如何监控应用等。从总体问题比例来看，夜莺服务端问题相对较少，大家摸索一下，很快可以掌握，疑问比较多的，是对各种目标的监控，比如如何监控 MySQL，如何监控 Redis 等，针对这部分问题，大家应该去查看采集器的文档，比如 Telegraf 每个采集器下面都有 [README介绍](https://github.com/influxdata/telegraf/tree/master/plugins/inputs)， 通过这些 README，理论上就可以知道如何使用各个采集器。Categraf 的采集器目录 [在这里](https://github.com/flashcatcloud/categraf/tree/main/inputs)， 未来我们也会把所有采集器补充完整的 README，以及告警规则和监控大盘JSON，大家导入直接就可以使用，当然，路漫漫其修远兮，一步一步来吧。