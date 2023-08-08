---
title : "高科技Startup构建监控体系之路"
description: "夜莺监控集成Grafana、Loki、Prometheus"
lead: ""
date: 2020-10-06T08:48:45+00:00
lastmod: 2020-10-06T08:48:45+00:00
draft: false
images: []  
weight: 1000
menu:
  docs:
    parent: "usecase"
toc: false
---


- [前言](#前言)
- [监控搭建](#监控搭建)
  - [夜莺搭建](#夜莺搭建)
  - [主机监控安装](#主机监控安装)
  - [BlackBox Exporter](#blackbox-exporter)
  - [Mysqld Exporter](#mysqld-exporter)
  - [consul + consul-template 动态生成配置](#consul--consul-template-动态生成配置)
    - [安装 Consul](#安装-consul)
    - [安装Consul-template](#安装consul-template)
    - [配置Consul K/V 动态生成URL监控](#配置consul-kv-动态生成url监控)
  - [修改Promtheus配置](#修改promtheus配置)
- [日志监控搭建](#日志监控搭建)
- [告警规则配置](#告警规则配置)
  - [系统运维](#系统运维)
    - [CPU利用率 > 90](#cpu利用率--90)
    - [Innode 利用率>90](#innode-利用率90)
    - [sshd 服务挂了](#sshd-服务挂了)
    - [内存利用率 > 95](#内存利用率--95)
    - [文件句柄 > 90](#文件句柄--90)
    - [IO wait > 30%](#io-wait--30)
    - [过去一分钟IOutil > 80](#过去一分钟ioutil--80)
    - [Ping > 1s](#ping--1s)
    - [平均负载>2](#平均负载2)
    - [TCP重传率>5%](#tcp重传率5)
    - [磁盘利用率 > 85%](#磁盘利用率--85)
    - [节点重启](#节点重启)
  - [业务运维](#业务运维)
    - [一分钟内日志ERROR>10](#一分钟内日志error10)
    - [URL探测不通](#url探测不通)
    - [过去一分钟出现Panic](#过去一分钟出现panic)
  - [数据库运维](#数据库运维)
    - [数据库重启](#数据库重启)
    - [连接数超过80%](#连接数超过80)
    - [最近一分钟有慢查询](#最近一分钟有慢查询)

> [贝联珠贯科技](https://www.lccomputing.com/), 一家ToB科技公司，定位于帮助客户大幅提升利用率，从而显著降低IT机器投入。

## 前言
公司当前机器总数100台左右, 没有监控, 总是在机器挂了才知道. 业务问题也只能依靠测试报障. 因为内部涉及多个K8s集群. 每个环境有独立的监控,日志收集系统, 所以需要一个`All IN ONE`的运维监控系统.  

尝试过`Grafana`+ `Mimir` + `Loki`的方式.二次开发成本过大, 并且短期内不能有效告警. 遂放弃. 接着尝试`夜莺V5`.

通过夜莺监控，免去了我们对告警通知的开发成本, 传统的 Grafana 或者 Alertmanager, 都需要二次对接自己的IM. 而夜莺支持了业务组或者部门的功能, 我们就可以利用这些功能做到告警细化, 并不需要再次对接IM平台. 并且有着更详细、易用的告警配置. 可以做到开箱即用, 学习成本近乎为零。

以下是实践过程，会从`系统运维`,`业务运维`,`数据库运维`等几个方面来进行监控系统搭建.

## 监控搭建

### 夜莺搭建

> <https://github.com/ccfos/nightingale>

这里选用最简单的`Docker Compose` 方式创建夜莺. 正如文档所说如果不是Docker专家, 不建议以这样的形式创建.
![夜莺文档](https://raw.githubusercontent.com/0x0034/images/master/20220615104159.png)

启动命令如下所示.

```powershell
git clone https://gitlink.org.cn/ccfos/nightingale.git
cd nightingale/docker
docker compose up -d
```

服务启动之后，浏览器访问nwebapi的端口，即18000，默认用户是`root`，密码是`root.2020`

### 主机监控安装

这里的主机监控agent 选用的`grafana-agent`, `grafana-agent` 集成了绝大部分会使用到的`exporter`, 做到了All IN ONE.  
并且支持Push 模式,简化流程, 这样在流程上只需要在主机启动时,预装`grafana-agent`, 由`grafana-agent`主动Push 到中心即可.

安装脚本如下所示:
> 这个脚本有如下几个注意点: 
>
> 1. `remote_write` 地址要根据自己部署夜莺的地址修改,将x.x.x.x更换为自己的IP即可
>
> 2. `$_hostip`: 这个建议写为主机IP, 因为对运维来说IP才是最直观的数据

```powershell
function InstallMonitor(){
  [ ! -f /usr/local/bin/grafana-agent  ] && wget -O /usr/local/bin/grafana-agent https://lcc-init.oss-cn-hangzhou-internal.aliyuncs.com/grafana-agent
  chmod +x /usr/local/bin/grafana-agent
  mkdir -p /metrics  /etc/grafana-agent
  cat >/etc/systemd/system/grafana-agent.service <<EOF
[Unit]
Description="grafana-agent"
After=network.target

[Service]
Type=simple

ExecStart=/usr/local/bin/grafana-agent -config.file /etc/grafana-agent/grafana-agent.yml
WorkingDirectory=/usr/local/bin

SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=grafana-agent
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
Restart=always


[Install]
WantedBy=multi-user.target

EOF
  chmod 0644 /etc/systemd/system/grafana-agent.service
  cat >/etc/grafana-agent/grafana-agent.yml <<EOF
server:
  log_level: info
  http_listen_port: 12345

metrics:
  wal_directory: /metrics

  global:
    scrape_interval: 15s
    scrape_timeout: 10s
    remote_write:
      # 远程写入的地址需要根据云上云下环境来切换.
      - url: http://x.x.x.x:19000/prometheus/v1/write
integrations:
  agent:
    enabled: true
  node_exporter:
    enabled: true
    instance: "$_hostip"
    include_exporter_metrics: true
  process_exporter:
    enabled: true
    instance: "$_hostip"
    process_names:
    - name: "{{.Comm}}"
      cmdline:
      - '.+'
EOF
  systemctl daemon-reload
  systemctl enable --now grafana-agent
}
```

### BlackBox Exporter

> 下载地址: <https://github.com/prometheus/blackbox_exporter/releases>

下载二进制文件并解压到`/usr/local/bin/`

安装脚本如下: 

```powershell
function InstallBlackboxExporter(){
  cat >/etc/systemd/system/blackbox_exporter.service <<EOF
[Unit]
Description="blackbox_exporter"
After=network.target

[Service]
Type=simple

ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox-exporter/blackbox.yml

WorkingDirectory=/usr/local/bin


SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=blackbox_exporter
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
Restart=always


[Install]
WantedBy=multi-user.target
EOF
  chmod 0644 /etc/systemd/system/blackbox_exporter.service
  cat >/etc/blackbox-exporter/blackbox.yml <<EOF
modules:
  http_2xx:
    prober: http
  http_post_2xx:
    prober: http
    http:
      method: POST
  tcp_connect:
    prober: tcp
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  grpc:
    prober: grpc
    grpc:
      tls: true
      preferred_ip_protocol: "ip4"
  grpc_plain:
    prober: grpc
    grpc:
      tls: false
      service: "service1"
  ssh_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
      - send: "SSH-2.0-blackbox-ssh-check"
  irc_banner:
    prober: tcp
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
EOF
  systemctl daemon-reload
  systemctl enable --now blackbox_exporter
}
```

### Mysqld Exporter

> 下载地址: <https://github.com/prometheus/mysqld_exporter>

下载二进制文件并解压到`/usr/local/bin/`

需要监听的数据库执行如下SQL:

> xxxxx替换为你设定的密码

```SQL
create user 'exporter'@'%' identified by 'xxxxx';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%'  WITH MAX_USER_CONNECTIONS 3;
flush privileges;

```

安装脚本如下: 

> `mysqld_exporter.cnf`: 中密码账户为上面执行SQL创建的用户密码.

```powershell
function InstallMysqldExporter(){
  cat >/etc/systemd/system/mysqld_exporter.service <<EOF
[Unit]
Description="mysqld_exporter"
After=network.target

[Service]
Type=simple

ExecStart=/usr/local/bin/mysqld_exporter --config.my-cnf=/etc/mysqld_exporter.cnf --collect.auto_increment.columns --collect.binlog_size --collect.global_status --collect.global_variables --collect.info_schema.innodb_metrics --collect.info_schema.innodb_cmp --collect.info_schema.innodb_cmpmem --collect.info_schema.processlist --collect.info_schema.query_response_time --collect.info_schema.tables --collect.info_schema.tablestats --collect.info_schema.userstats --collect.perf_schema.eventswaits --collect.perf_schema.file_events --collect.perf_schema.indexiowaits --collect.perf_schema.tableiowaits --collect.perf_schema.tablelocks --collect.slave_status

WorkingDirectory=/usr/local/bin


SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=mysqld_exporter
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
Restart=always


[Install]
WantedBy=multi-user.target
EOF
  chmod 0644 /etc/systemd/system/mysqld_exporter.service
  cat >/etc/mysqld_exporter.cnf <<EOF
[client]
user=exporter
password=xxxx
host=x.x.x.x
port=3306
EOF
  systemctl daemon-reload
  systemctl enable --now mysqld_exporter
}
```

### consul + consul-template 动态生成配置

#### 安装 Consul

> `-bind` 和 `-client` 需要替换为本机IP

```powershell
function InstallConsul(){
  yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
  yum -y install consul 
  mkdir -p /data/consul
  cat >/etc/systemd/system/consul.service <<EOF
[Unit]
Description="consul"
After=network.target

[Service]
Type=simple

ExecStart=/usr/bin/consul agent -server -bootstrap-expect 1 -bind=x.x.x.x -client=x.x.x.x -data-dir=/data/consul -node=agent-one -config-dir=/etc/consul.d -ui

WorkingDirectory=/usr/bin/


SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=consul
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
Restart=always


[Install]
WantedBy=multi-user.target
EOF
  chmod 0644 /etc/systemd/system/consul.service
  systemctl daemon-reload
  systemctl enable --now consul
}
```

#### 安装Consul-template

安装脚本如下所示:

> `x.x.x.x` 替换为夜莺地址 , `a.b.c.d` 替换为consul部署地址

```powershell
wget https://releases.hashicorp.com/consul-template/0.29.0/consul-template_0.29.0_linux_amd64.zip
unzip consul-template_0.29.0_linux_amd64.zip
chmod +x consul-template
mv consul-template /usr/local/bin/consul-template
mkdir -p /etc/consul-template/template
cat > /etc/consul-template/consul-template.conf << EOF
log_level = "warn"
syslog {
  # This enables syslog logging.
  enabled = true
  # This is the name of the syslog facility to log to.
  facility = "LOCAL5"
}
consul {
  # auth {
    # enabled = true
    # username = "test"
  # password = "test"
  # }
  # 注意替换为consul地址
  address = "a.b.c.d:8500"
  retry {
    enabled = true
    attempts = 12
    backoff = "250ms"
    # If max_backoff is set to 10s and backoff is set to 1s, sleep times
    # would be: 1s, 2s, 4s, 8s, 10s, 10s, ...
    max_backoff = "3m"
  }
}
template {
  source = "/etc/consul-template/templates/url-monitor.ctmpl"
  destination = "/home/nightingale-main/docker/prometc/conf.d/url/url.yaml"
  command = "curl -X POST http://x.x.x.x:9090/-/reload"
  command_timeout = "60s"
  backup = true
  wait {
    min = "2s"
    max = "20s"
  }
}
template {
  source = "/etc/consul-template/templates/icmp-monitor.ctmpl"
  destination = "/home/nightingale-main/docker/prometc/conf.d/icmp/icmp.yaml"
  command = ""
  command_timeout = "60s"
  backup = true
  wait {
    min = "2s"
    max = "20s"
  }
}
EOF

cat > /etc/consul-template/consul-template.conf/template/url-monitor.ctmpl <<EOF
- targets:
{{- range ls  "blackbox/url/http200" }}
  - http://{{ .Key  }}{{ .Value }}
{{- end }}
EOF

cat > /etc/consul-template/consul-template.conf/template/icmp-monitor.ctmpl <<EOF
{{- range ls  "blackbox/icmp" }}
- targets:
  - {{ .Key  }}
  labels:
    instance: {{ .Key }}
{{- end }}
EOF



cat > /etc/systemd/system/consul-template.service <<EOF
[Unit]
Description="consul-template"
After=network.target

[Service]
Type=simple

ExecStart=/usr/local/bin/consul-template -config /etc/consul-template/consul-template.conf

WorkingDirectory=/usr/local/bin


SuccessExitStatus=0
LimitNOFILE=65536
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=consul-template
KillMode=process
KillSignal=SIGQUIT
TimeoutStopSec=5
Restart=always


[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable --now consul-template.service
```

#### 配置Consul K/V 动态生成URL监控

添加如下`K/V`,`K/V` 对应上文`*.ctmpl` 文件中渲染地址. 在这里`Key` 为域名,`Values` 为路径
![Conusl配置](https://raw.githubusercontent.com/0x0034/images/master/20220615113931.png)

### 修改Promtheus配置

`nightingale-main/docker/prometc/prometheus.yml`追加如下内容:

```yaml
  - job_name: MySQL
    static_configs:
    - targets:
      - x.x.x.x:9104
      labels:
        instance: MySQL-dev
  - job_name: process
    static_configs:
    - targets:
      - x.x.x.x:9256
  - job_name: 'blackbox-url-monitor'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    file_sd_configs:
    - refresh_interval: 1m
      files:
        - ./conf.d/url/*.yaml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: x.x.x.x:9115
  - job_name: 'blackbox-icmp-monitor'
    scrape_interval: 1m
    metrics_path: /probe
    params:
      module: [icmp]
    file_sd_configs:
    - refresh_interval: 1m
      files:
        - ./conf.d/icmp/*.yaml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: x.x.x.x:9115
```

在`nightingale-main/docker/prometc/` 下创建目录`conf.d`. 命令如下:

```powershell
cd nightingale-main/docker/prometc/ 
mkdir -p conf.d/{icmp,url}
```

重启promtheus,命令如下所示:

```powershell
docker restart prometheus
```

重启后检查prometheus状态
![promtheus状态](https://raw.githubusercontent.com/0x0034/images/master/20220615115009.png)

## 日志监控搭建

> 感谢夜莺社区支持.
>
> 1. 大前提, 夜莺版本高于5.9.2
> 2. 已有Loki. 并且Loki已经支持多租户.

Loki的配置在这里不做赘述,网上教程太多了.

`docker-compose.yml` 追加如下内容, 与`nserver` 同级

```yaml
  lokinserver:
    image: registry.cn-hangzhou.aliyuncs.com/lcc-middleware/nightingale:5.9.2
    container_name: lokinserver
    hostname: nserver
    restart: always
    environment:
      GIN_MODE: release
      TZ: Asia/Shanghai
      WAIT_HOSTS: mysql:3306, redis:6379
    volumes:
      - ./lokin9eetc:/app/etc
    ports:
      - "20000:20000"
    networks:
      - nightingale
    depends_on:
      - mysql
      - redis
      - prometheus
      - ibex
    links:
      - mysql:mysql
      - redis:redis
      - prometheus:prometheus
      - ibex:ibex
    command: >
      sh -c "/wait && /app/n9e server"
```

生成`lokinserver`容器的配置文件.操作如下.

```powershell
cp -r n9eetc lokin9eetc
cd lokin9eetc
```

修改`lokin9eetc/server.conf`文件中`Reader`字段,内容如下:

> 如果开启多租户记得传`Headers`, 如果没开,则去除`Headers`字段
> Loki的API中带`loki`前缀的都是兼容`prometheus`风格的API 所以一定要加. `Prom`字段替换为自己的域名

```toml
[Reader]
# prometheus base url
Url = "http://loki.xxx.xxx/loki/"
# Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 10
Headers = ["X-Scope-OrgID","lcc-loki"]
```

修改配置文件`nightingale-main/docker/n9eetc/webapi.conf`, 追加如下内容

> 如果开启多租户记得传`Headers`, 如果没开,则去除`Headers`字段
> Loki的API中带`loki`前缀的都是兼容`prometheus`风格的API 所以一定要加. `Prom`字段替换为自己的域名

```toml
[[Clusters]]
# Prometheus cluster name
Name = "Loki"
# # Prometheus APIs base url
Prom = "http://loki.xxx.xxx/loki/"
# # Basic auth username
BasicAuthUser = ""
# Basic auth password
BasicAuthPass = ""
# timeout settings, unit: ms
Timeout = 30000
DialTimeout = 10000
TLSHandshakeTimeout = 30000
ExpectContinueTimeout = 1000
IdleConnTimeout = 90000
# time duration, unit: ms
KeepAlive = 30000
MaxConnsPerHost = 0
MaxIdleConns = 100
MaxIdleConnsPerHost = 100
Headers = ["X-Scope-OrgID","lcc-loki"]
```

重启夜莺监控:

```powershell
docker-compose up -d
```

## 告警规则配置

### 系统运维

#### CPU利用率 > 90

```PromQL
(100-(avg by (mode, instance)(rate(node_cpu_seconds_total{mode="idle"}[1m])))*100) > 90
```

#### Innode 利用率>90

```PromQL
(100 - ((node_filesystem_files_free * 100) / node_filesystem_files))>90
```

#### sshd 服务挂了

```PromQL
(namedprocess_namegroup_num_procs{groupname="sshd"}) == 0
```

#### 内存利用率 > 95

```PromQL
(node_memory_MemTotal_bytes - node_memory_MemFree_bytes - (node_memory_Cached_bytes + node_memory_Buffers_bytes))/node_memory_MemTotal_bytes*100 > 95
```

#### 文件句柄 > 90

```PromQL
(node_filefd_allocated{}/node_filefd_maximum{}*100)
```

#### IO wait > 30%

```PromQL
avg by (instance) (rate(node_cpu_seconds_total{mode="iowait"}[5m])) * 100 > 30
```

#### 过去一分钟IOutil > 80

```PromQL
(rate(node_disk_io_time_seconds_total{} [1m]) *100) > 80
```

#### Ping > 1s

```PromQL
avg_over_time(probe_icmp_duration_seconds[1m]) > 1
```

#### 平均负载>2

```PromQL
(avg(node_load1) by(instance)/count by (instance)(node_cpu_seconds_total{mode='idle'})) >2 
```

#### TCP重传率>5%

```PromQL
(rate(node_netstat_Tcp_RetransSegs{}[5m])/ rate(node_netstat_Tcp_OutSegs{}[5m])*100)  > 5 
```

#### 磁盘利用率 > 85%

```PromQL
(100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes) ) > 85
```

#### 节点重启

```PromQL
node_reboot_required > 0
```

### 业务运维

> 我们是GO应用,其他应用根据需要设定

#### 一分钟内日志ERROR>10

> 日志这里主要选,我们上面添加的Loki集群

![error日志](https://raw.githubusercontent.com/0x0034/images/master/20220615124307.png)

#### URL探测不通

```PromQL
probe_http_status_code <= 199 OR probe_http_status_code >= 400
```

#### 过去一分钟出现Panic

![Panic日志](https://raw.githubusercontent.com/0x0034/images/master/20220615124533.png)

### 数据库运维

> 仅罗列部分, 更多可以在导入规则中查找

![mysql规则](https://raw.githubusercontent.com/0x0034/images/master/20220615124907.png)

#### 数据库重启

```PromQL
mysql_global_status_uptime < 60
```

#### 连接数超过80%

```PromQL
avg by (instance) (mysql_global_status_threads_connected) / avg by (instance) (mysql_global_variables_max_connections) * 100 > 80`
```

#### 最近一分钟有慢查询

```PromQL
increase(mysql_global_status_slow_queries[1m]) > 0
```