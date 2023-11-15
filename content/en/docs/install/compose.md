---
title: "Docker Compose"
description: "Deploy nightingale via Docker compose"
lead: ""
date: 2020-11-12T13:26:54+01:00
lastmod: 2020-11-12T13:26:54+01:00
draft: false
images: []
menu:
  docs:
    parent: "install"
weight: 610
toc: true
---


```bash
$ git clone https://gitlink.org.cn/ccfos/nightingale.git
$ cd nightingale/docker/compose-bridge

$ docker-compose up -d
Creating network "docker_nightingale" with driver "bridge"
Creating mysql      ... done
Creating redis      ... done
Creating prometheus ... done
Creating ibex       ... done
Creating agentd     ... done
Creating nwebapi    ... done
Creating nserver    ... done
Creating telegraf   ... done

$ docker-compose ps
NAME                IMAGE                              COMMAND                  SERVICE             CREATED             STATUS              PORTS
categraf            flashcatcloud/categraf:latest      "/entrypoint.sh"         categraf            2 days ago          Up 2 days
ibex                ulric2019/ibex:0.3                 "sh -c '/wait && /ap…"   ibex                2 days ago          Up 2 days
mysql               mysql:5.7                          "docker-entrypoint.s…"   mysql               2 days ago          Up 2 days
n9e                 flashcatcloud/nightingale:latest   "sh -c '/wait && /ap…"   n9e                 2 days ago          Up 2 days
prometheus          prom/prometheus                    "/bin/prometheus --c…"   prometheus          2 days ago          Up 2 days
redis               redis:6.2                          "docker-entrypoint.s…"   redis               2 days ago          Up 2 days
```

Use your browser to access Nightingale's web page at [http://localhost:17000](http://localhost:17000). The default username is root and default password is `root.2020`.



