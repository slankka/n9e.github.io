---
title: "Introduction"
description: "An enterprise-level cloud-native monitoring system, which can be used as drop-in replacement of Prometheus for alerting and Grafana for visualization."
date: 2020-10-06T08:48:57+00:00
lastmod: 2020-10-06T08:48:57+00:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 100
toc: true
---

Nightingale project is an enterprise-level cloud-native monitoring system, which can be used as drop-in replacement of Prometheus for alerting and Grafana for visualization.

## Repo

- Backend: [https://github.com/ccfos/nightingale](https://github.com/ccfos/nightingale)
- Frontend: [https://github.com/n9e/fe](https://github.com/n9e/fe)

Any issues or PRs are welcome!

## Architecture

![20230808114159](https://download.flashcat.cloud/ulric/20230808114159.png)

The Nightingale project is very open and can interface with common collectors in the open source community, such as categraf, telegraf, datadog-agent, grafana-agent, as well as common time series databases in the open source community, such as Prometheus, VictoriaMetrics, Thanos, as well as logging stores, such as ElasticSearch, Loki, as well as common notification mediums, such as Slack, mm, Dingtalk, Wecom.
