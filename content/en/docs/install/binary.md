---
title: "Binary install"
description: "Deploy nightingale via binary"
lead: ""
date: 2020-11-12T13:26:54+01:00
lastmod: 2020-11-12T13:26:54+01:00
draft: false
images: []
menu:
  docs:
    parent: "install"
weight: 630
toc: true
---

## Download

Download the latest release from [Github](https://github.com/ccfos/nightingale/releases)

## Install

```bash
#!/bin/bash
mkdir /opt/n9e && tar zxvf n9e-v6.0.1-linux-amd64.tar.gz -C /opt/n9e

cd /opt/n9e

# init database
mysql -uroot -p1234 < n9e.sql

# check configurations in /opt/n9e/etc/config.toml
nohup ./n9e &> n9e.log &
```

## Check Process

```bash
ss -tlnp|grep 17000
```

## Login

[http://localhost:17000](http://localhost:17000)。Default username is root and default password is `root.2020`.


