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
$ git clone https://github.com/ccfos/nightingale.git
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

Use your browser to access Nightingale's web page at [http://localhost:17000](http://localhost:17000). The default username is `root` and default password is `root.2020`.

## Security
To change passwords for security reasons, it's recommand to change passwords before installing. Don't worry, we still have chance to do after initialization.

### change passwords before installing
**change mysql password before installing and initialization**
* change c-init.sql
* change ./docker/compose-bridge/docker-compose.yaml `MYSQL_ROOT_PASSWORD: xxxx`
* change ./docker/compose-bridge/etc-nightingale/config.toml, go to `[DB]` part, and set `DSN = root:xxxx@tcp(mysql:3306)....`

**change Redis password**
It's both ok for first time launch or non-first time launch.

* `cd ./docker/compose-bridge/ && mkdir -p ./docker/etc-redis/ && echo 'requirepass xxxx' ./etc-redis/redis.conf`
* change docker-compose.yaml to mount redis.conf
* change ./docker/compose-bridge/etc-nightingale/config.toml, go to `[Redis]` part, and set `Password = xxxx`

```
    volumes:
      - ./etc-redis/redis.conf:/etc/redis/redis.conf
```

**change nightingale default password before installing**
* change ./docker/initsql/a-n9e.sql 'insert into `users` ...'

### change passwords for exists docker deployments

**change mysql password for exists mysqldata** 
* `docker exec -it mysql /bin/bash`, login with default mysql password then execute `ALTER TABLE mysql.user 'root'%'127.0.0.1' IDENTIFIED BY xxxx`
* change mysql password configuration then restart docker compose containers.

**change nightingale password for exists containers and mysqldata**
* go to nightingale UI `http://localhost:17000/account/profile/pwd` and reset password, it takes effect for data persists in `./docker/compose-bridge/mysqldata`

