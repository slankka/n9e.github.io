---
title: "Grafana-Agent"
description: "Use Grafana-Agent as collector for Nightingale"
lead: ""
date: 2022-05-12T13:26:54+01:00
lastmod: 2022-05-12T13:26:54+01:00
draft: false
images: []
menu:
  docs:
    parent: "agent"
weight: 730
toc: true
---

## Introduction

Grafana Agent is a vendor-neutral, batteries-included telemetry collector with configuration inspired by Terraform. It is designed to be flexible, performant, and compatible with multiple ecosystems such as Prometheus and OpenTelemetry.

Grafana Agent is based around components. Components are wired together to form programmable observability pipelines for telemetry collection, processing, and delivery.

## Configuration

```yaml
server:
  log_level: info
  http_listen_port: 12345

metrics:
  global:
    scrape_interval: 15s
    scrape_timeout: 10s
    remote_write:
      - url: https://N9E:17000/prometheus/v1/write
        basic_auth:
          username: ""
          password: ""

integrations:
  agent:
    enabled: true

  node_exporter:
    enabled: true
    include_exporter_metrics: true
```

## Restart

```bash
nohup ./agent-linux-amd64 \
  -config.file ./agent-cfg.yaml \
  -metrics.wal-directory ./data \
  &> grafana-agent.log &
```