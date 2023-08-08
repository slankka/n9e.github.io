---
title: "Categraf"
description: "use categraf to collect metrics"
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

[Categraf](https://github.com/flashcatcloud/categraf) is an agent which can collect metrics and logs. Categraf uses `prometheus remote write` as data push protocol, so it can push metrics to Nightingale.

## Configuration

Configuration file of categraf: `conf/config.toml`

```toml
[writer_opt]
# default: 2000
batch = 2000
# channel(as queue) size
chan_size = 10000

[[writers]]
url = "http://N9E:17000/prometheus/v1/write"

# Basic auth username
basic_auth_user = ""

# Basic auth password
basic_auth_pass = ""

# timeout settings, unit: ms
timeout = 5000
dial_timeout = 2500
max_idle_conns_per_host = 100

[heartbeat]
enable = true

# report os version cpu.util mem.util metadata
url = "http://N9E:17000/v1/n9e/heartbeat"

# interval, unit: s
interval = 10

# Basic auth username
basic_auth_user = ""

# Basic auth password
basic_auth_pass = ""

## Optional headers
# headers = ["X-From", "categraf", "X-Xyz", "abc"]

# timeout settings, unit: ms
timeout = 5000
dial_timeout = 2500
max_idle_conns_per_host = 100
```

We highly recommend that you use Categraf as collector for Nightingale.
