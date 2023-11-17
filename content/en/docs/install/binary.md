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

Download the latest release from [GitHub](https://github.com/ccfos/nightingale/releases), then you will get a tarball which name is like `n9e-{version}-linux-amd64.tar.gz`

## Install

```bash
#!/bin/bash
mkdir /opt/n9e && tar zxvf n9e-{version}-linux-amd64.tar.gz -C /opt/n9e

cd /opt/n9e

# init database
mysql -uroot -p1234 < n9e.sql

# check configurations in /opt/n9e/etc/config.toml and start n9e
nohup ./n9e &> n9e.log &
```

## Check Process

```bash
# check process is runing or not
ss -tlnp|grep 17000
```

## Login

Open web browser and go to [http://localhost:17000](http://localhost:17000). The default username is `root` and default password is `root.2020`.