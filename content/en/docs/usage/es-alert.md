---
title: "基于夜莺快速构建日志告警平台"
description: "基于夜莺快速构建日志告警平台"
lead: ""   
date: 2022-10-03T10:55:57+00:00
lastmod: 2022-10-03T10:55:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "usage"
weight: 830
toc: true
---

在收集到日志之后，我们通常会有下面几类基于日志做告警的需求：

- 统计日志中的 ERROR 关键字出现次数，超过阈值则发出告警；
- 从网关日志中提取服务接口的 QPS，出现较大波动则发出告警；
- 从网关日志中提取服务接口的延迟，延迟太高则发出告警；

我们可以基于夜莺的的整体流程，快速构建一个日志告警平台来满足上述的需求，本文从产品设计，架构设计，代码实现三个方面，来介绍如何基于夜莺来构建日志告警平台。为了满足大家的好奇心，我们来先看一张效果图，下图是告警历史详情页的截图。

![告警历史详情](https://blog-1255977231.cos.ap-beijing.myqcloud.com/uPic/image-20220924161828240.png)

## 产品设计

通过前文的介绍，我们的需求已经比较明确了，将应用的日志收集起来，在平台上配置告警规则，根据配置的规则触发告警通知，收到通知之后，我们在页面上可以看到和告警数据相关的日志原文，便于快速定位问题。

夜莺监控已经支持了告警规则配置页面和告警详情查看页面，所以我们可以直接复用这两个页面。当收到告警通知，点击到告警详情页时，夜莺目前的告警详情页只有查看时序数据的能力，没有查看日志原文的能力，所以还需要增加在详情页查看日志原文的能力。所以为了支持日志告警能力，在产品层面我们需要对夜莺的两个页面（告警规则页面、告警历史详情页面）进行改造。

![告警规则原型设计](https://blog-1255977231.cos.ap-beijing.myqcloud.com/uPic/image-20220924155106962.png)

![告警历史原型设计](https://blog-1255977231.cos.ap-beijing.myqcloud.com/uPic/image-20220924155038720.png)


## 架构设计

产品设计确定之后，我们来进行架构设计，首先看一下夜莺已有的能力

- n9e-webapi 提供各种配置管理、即时数据查询、告警历史详情查看的能力
- n9e-server 提供同步规则、查询数据、产生异常点、生成告警事件、告警屏蔽、告警订阅、告警通知的能力。

如果要增加日志告警能力，我们可以发现在产生异常点之后，n9e-server 后续的一系列能力是可以复用的，所以我们可以得到下面的架构图

![架构图](https://blog-1255977231.cos.ap-beijing.myqcloud.com/uPic/image-20220924153708382.png)

从上图我们可以发现，日志告警模块只要实现下面4个功能即可

1. 同步告警规则
2. 从日志中提取 metric 数据和原文
3. 根据规则判断数据是否异常
4. 异常点发送给 n9e-server

搞清楚要做什么事情之后，下面我们要开始动手写代码了

## 代码实现

本小节主要是介绍日志告警模块代码实现的思路，我们可以开发一个独立的模块，主要是实现下面四个功能。

##### 同步告警规则

我们可以通过定期查询 n9e-webapi 提供的 api 来同步告警规则，同步逻辑可以参考 n9e-server 模块 https://github.com/ccfos/nightingale/blob/main/src/server/memsto/alert_rule_cache.go 的逻辑，将从数据库查询，改成从 api 获取即可

##### 日志中查询 metric 数据和原文

Elasticsearch 提供了 [bucket aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html "bucket aggregations") 和 [metric aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html "metric aggregations") 的 api，通过这两个 api ，我们可以根据查询条件查到对应的 metric 数据。通过 es 的 search api，我们可以根据查询条件查到日志原文。

##### 根据规则判断数据是否异常

这个功能可以参考 n9e-server  的告警规则判断逻辑，代码在 https://github.com/ccfos/nightingale/blob/main/src/server/engine/worker.go  loopFilterRules() 定期获取规则，然后生成异常点检测任务，Work() 实现了产生异常点的功能。

##### 异常点发送给 n9e-server

n9e-server 提供了接收异常点然后产生告警事件的接口，产生异常点之后，我们把异常点 push 给 n9e-server 的 [api](https://github.com/ccfos/nightingale/blob/main/src/server/router/router.go#L106 "api") 即可，之后的告警事件处理流程，全部由 n9e-server 来负责。

## 最终效果

通过夜莺日志告警插件，我们可以在夜莺平台，实现日志场景下的监控告警体系建设。

![产品截图](https://blog-1255977231.cos.ap-beijing.myqcloud.com/uPic/image-20220924193311484.png)

> **One more thing**

快猫星云技术团队已经实现了夜莺日志告警这个模块，如果您在使用夜莺监控，遇到了 Metric 监控覆盖不到的场景，需要对日志进行监控告警，可以点击[链接](https://sourl.cn/7JTAV8)购买试用，目前正在打折优惠，团队版首次试用费用只需 **29** 元， 感兴趣的可以买起来：）