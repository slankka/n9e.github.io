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

<img src="/images/intro/arch-simple.png" />
<br />
<br />

The Nightingale project is very open and can interface with common collectors in the open source community, such as categraf, telegraf, datadog-agent, grafana-agent, as well as common time series databases in the open source community, such as Prometheus, VictoriaMetrics, Thanos, as well as logging stores, such as ElasticSearch, Loki, as well as common notification mediums, such as Slack, mm, Dingtalk, Wecom.

## Key Capabilities

<img src="/images/intro/rule-config.png" />
<br />
<br />

Nightingle can be used as an alert engine to make anomaly judgment on data. It supports to configure different effective time for alert strategies, multiple judgment rules can be configured within the same alert strategy, and multiple rules can be inhibited according to the severity.

<img src="/images/intro/dashboard-black.png" />
<br />
<br />

Nightingale can be used as a visualization tool, similar to Grafana, to view metrics and log data, and supports making dashboards, which support pie charts, line charts, and many other chart types.

<img src="/images/intro/integrations.png" />
<br />
<br />

Nightingale has built-in alerting rules and dashboards for different middleware and databases right out of the box.

<img src="/images/intro/elasticsearch-index-patterns.png" />
<br />
<br />

Nightingale supports Kibana-like query exploration, which allows you to filter logs by keywords and filters.


