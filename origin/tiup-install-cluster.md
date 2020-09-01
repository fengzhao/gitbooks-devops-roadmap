# 使用TIUP部署TiDB 4.0集群

# 一、简介

[TiUP](https://github.com/pingcap/tiup) 是 TiDB 4.0 版本引入的集群运维工具，[TiUP cluster](https://github.com/pingcap/tiup/tree/master/components/cluster) 是 TiUP 提供的使用 Golang 编写的集群管理组件，通过 TiUP cluster 组件就可以进行日常的运维工作，包括部署、启动、关闭、销毁、弹性扩缩容、升级 TiDB 集群；管理 TiDB 集群参数。

目前 TiUP 可以支持部署 TiDB、TiFlash、TiDB Binlog、TiCDC，以及监控系统。

## Linux 操作系统版本要求

| Linux 操作系统平台       | 版本         |
| ------------------------ | ------------ |
| Red Hat Enterprise Linux | 7.3 及以上   |
| CentOS                   | 7.3 及以上   |
| Oracle Enterprise Linux  | 7.3 及以上   |
| Ubuntu LTS               | 16.04 及以上 |

## TiDB各组件端口

| 组件              | 默认端口 | 说明                                               |
| ----------------- | -------- | -------------------------------------------------- |
| TiDB              | 4000     | 应用及 DBA 工具访问通信端口                        |
| TiDB              | 10080    | TiDB 状态信息上报通信端口                          |
| TiKV              | 20160    | TiKV 通信端口                                      |
| TiKV              | 20180    | TiKV 状态信息上报通信端口                          |
| PD                | 2379     | 提供 TiDB 和 PD 通信端口                           |
| PD                | 2380     | PD 集群节点间通信端口                              |
| TiFlash           | 9000     | TiFlash TCP 服务端口                               |
| TiFlash           | 8123     | TiFlash HTTP 服务端口                              |
| TiFlash           | 3930     | TiFlash RAFT 服务和 Coprocessor 服务端口           |
| TiFlash           | 20170    | TiFlash Proxy 服务端口                             |
| TiFlash           | 20292    | Prometheus 拉取 TiFlash Proxy metrics 端口         |
| TiFlash           | 8234     | Prometheus 拉取 TiFlash metrics 端口               |
| Pump              | 8250     | Pump 通信端口                                      |
| Drainer           | 8249     | Drainer 通信端口                                   |
| CDC               | 8300     | CDC 通信接口                                       |
| Prometheus        | 9090     | Prometheus 服务通信端口                            |
| Node_exporter     | 9100     | TiDB 集群每个节点的系统信息上报通信端口            |
| Blackbox_exporter | 9115     | Blackbox_exporter 通信端口，用于 TiDB 集群端口监控 |
| Grafana           | 3000     | Web 监控服务对外服务和客户端(浏览器)访问端口       |
| Alertmanager      | 9093     | 告警 web 服务端口                                  |
| Alertmanager      | 9094     | 告警通信端口                                       |



# 二、集群节点准备

## 0、集群组件规划

| 集群角色/集群节点 | tools.tidb4.curiouser.com | node1.tidb4.curiouser.com | node2.tidb4.curiouser.com | node3.tidb4.curiouser.com |
| ----------------- | :-----------------------: | :-----------------------: | :-----------------------: | :-----------------------: |
| TiUP              |             ✓             |                           |                           |                           |
| TiDP              |                           |             ✓             |             ✓             |             ✓             |
| PD Server         |                           |             ✓             |             ✓             |             ✓             |
| TiKV              |                           |             ✓             |             ✓             |             ✓             |
| TiFlash           |             ✓             |                           |                           |                           |
| CDC               |             ✓             |                           |                           |                           |
| pump              |             ✓             |                           |                           |                           |
| grafana           |             ✓             |                           |                           |                           |
| prometheus        |             ✓             |                           |                           |                           |
| alertmanager      |             ✓             |                           |                           |                           |
| Zookeeper         |             ✓             |                           |                           |                           |
| Kafka             |             ✓             |                           |                           |                           |

## 1、节点准备

### ①所有节点创建tidb用户并加入sudoer

```bash
useradd -m tidb
echo "tidb ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
passwd tidb
```

### ②所有节点关闭系统 swap

```bash
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a
sysctl -p
```

### ③打通tools节点的tidb用户到所有节点tidb用户的免秘钥登录

```bash
su - tidb 
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
ssh-copy-id -i tools.tidb4.curiouser.com
ssh-copy-id -i node1.tidb4.curiouser.com
ssh-copy-id -i node2.tidb4.curiouser.com
ssh-copy-id -i node3.tidb4.curiouser.com
```



# 三、集群安装

以下所有命令都在tools节点，tidb用户，bash环境下执行（zsh环境不支持tiup的某些脚本）

## 1、安装tiup命令

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
source .bash_profile
tiup cluster
tiup --binary cluster
```

## 2、查看目前的tiup命令支持安装的tidb版本

```bash
tiup list tidb
```

## 3、创建集群组件的拓扑配置文件

```yaml
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/data/tidb/bin"
  data_dir: "/data/tidb/data"
  arch: "amd64"
  resource_control:
    memory_limit: "6G"
    cpu_quota: "200%"
monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
  deploy_dir: "/data/tidb/bin/monitored-9100"
  data_dir: "/data/tidb/data/tidb-data-monitored-9100"
  log_dir: "/data/tidb/logs/monitored-9100"

server_configs:
  tidb:
    log.slow-threshold: 300
    binlog.enable: true
    binlog.ignore-error: true
  tikv:
    readpool.unified.max-thread-count: 12
    readpool.storage.use-unified-pool: false
    readpool.coprocessor.use-unified-pool: true
    storage.block-cache.capacity: "16GB"
    raftstore.capacity: "10GB"
  pd:
    replication.location-labels: ["host"]
    schedule.leader-schedule-limit: 4
    schedule.region-schedule-limit: 2048
    schedule.replica-schedule-limit: 64
  tiflash:
    logger.level: "info"
    # Maximum memory usage for processing a single query. Zero means unlimited.
    profiles.default.max_memory_usage: 10000000000
    # Maximum memory usage for processing all concurrently running queries on the server. Zero means unlimited.
    profiles.default.max_memory_usage_for_all_queries: 0

pd_servers:
  - host: 192.168.1.71
  - host: 192.168.1.72
  - host: 192.168.1.73

tidb_servers:
  - host: 192.168.1.71
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/bin/tidb-4000"
    log_dir: "/data/tidb/logs/tidb-4000"
    numa_node: "0"
  - host: 192.168.1.72
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/bin/tidb-4000"
    log_dir: "/data/tidb/logs/tidb-4000"
    numa_node: "0"
  - host: 192.168.1.73
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/bin/tidb-4000"
    log_dir: "/data/tidb/logs/tidb-4000"
    numa_node: "0"

tikv_servers:
  - host: 192.168.1.71
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/bin/tikv-20160"
    data_dir: "/data/tidb/data/tikv-20160"
    log_dir: "/data/tidb/logs/tikv-20160"
    numa_node: "0"
    config:
      server.labels: { host: "tikv1" }
  - host: 192.168.1.72
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/bin/tikv-20160"
    data_dir: "/data/tidb/data/tikv-20160"
    log_dir: "/data/tidb/logs/tikv-20160"
    numa_node: "0"
    config:
      server.labels: { host: "tikv2" }
  - host: 192.168.1.73
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/bin/tikv-20160"
    data_dir: "/data/tidb/data/tikv-20160"
    log_dir: "/data/tidb/logs/tikv-20160"
    numa_node: "0"
    config:
      server.labels: { host: "tikv3" }
tiflash_servers:
  - host: 192.168.1.70
    data_dir: "/data/tidb/data/tiflash-9000"
    deploy_dir: "/data/tidb/bin/tiflash-9000"
    ssh_port: 22
    tcp_port: 9000
    http_port: 8123
    flash_service_port: 3930
    flash_proxy_port: 20170
    flash_proxy_status_port: 20292
    metrics_port: 8234
    numa_node: "0"
    # The following configs are used to overwrite the `server_configs.tiflash` values.
    config:
      logger.level: "info"
    learner_config:
      log-level: "info"
pump_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 8250
    deploy_dir: "/data/tidb/bin/pump-8249"
    data_dir: "/data/tidb/data/pump-8249"
    log_dir: "/data/tidb/logs/pump-8249"
    numa_node: "0"
    # The following configs are used to overwrite the `server_configs.drainer` values.
    config:
      gc: 7
tispark_masters:
  - host: 192.168.1.70
    ssh_port: 22
    port: 7077
    web_port: 8080
    deploy_dir: "/data/tidb/bin/tispark-master-7077"
    java_home: "/opt/jdk"
    spark_config:
      spark.driver.memory: "2g"
      spark.eventLog.enabled: "False"
      spark.tispark.grpc.framesize: 268435456
      spark.tispark.grpc.timeout_in_sec: 100
      spark.tispark.meta.reload_period_in_sec: 60
      spark.tispark.request.command.priority: "Low"
      spark.tispark.table.scan_concurrency: 256
    spark_env:
      SPARK_EXECUTOR_CORES: 5
      SPARK_EXECUTOR_MEMORY: "10g"
      SPARK_WORKER_CORES: 5
      SPARK_WORKER_MEMORY: "10g"

# NOTE: multiple worker nodes on the same host is not supported by Spark
tispark_workers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 7078
    web_port: 8081
    deploy_dir: "/data/tidb/bin//tispark-worker-7078"
    java_home: "/opt/jdk"

cdc_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 8300
    deploy_dir: "/data/tidb/bin/cdc-8300"
    log_dir: "/tidb-deploy/cdc-8300/log"

monitoring_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 9090
    deploy_dir: "/data/tidb/bin/prometheus-9090"
    data_dir: "/data/tidb/data/prometheus-9090"
    log_dir: "/data/tidb/logs/prometheus-9090"

grafana_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 3000
    deploy_dir: "/data/tidb/bin/grafana-3000"

alertmanager_servers:
  - host: 192.168.1.70
    ssh_port: 22
    web_port: 9093
    cluster_port: 9094
    deploy_dir: "/data/tidb/bin/alertmanager-9093"
    data_dir: "/data/tidb/data/alertmanager-9093"
    log_dir: "/data/tidb/logs/alertmanager-9093"
```

## 4、查集群拓扑配置文件语法

```bash
tiup cluster check 集群拓扑配置文件
```

## 5、分发组件文件到各节点

```bash
tiup cluster deploy 集群名字 tidb版本 ./集群拓扑配置.yaml --user tidb -y
```

## 6、所有节点安装`numactl`

在生产环境中，因为硬件机器配置往往高于需求，为了更合理规划资源，会考虑单机多实例部署 TiDB 或者 TiKV。NUMA 绑核工具的使用，主要为了防止 CPU 资源的争抢，引发性能衰退。NUMA 绑核是用来隔离 CPU 资源的一种方法，适合高配置物理机环境部署多实例使用。



通过 `tiup cluster deploy` 完成部署操作才可以通过 `exec` 命令来进行集群级别管理工作

```bash
tiup cluster exec 集群名字 --sudo --command "yum install -y numactl" 
# 或者
tiup cluster exec 集群名字 --sudo --command "yum install -y numactl" -R PD
# 或者
tiup cluster exec 集群名字 --sudo --command "yum install -y numactl" -N node1
```

## 7、启动当前集群各组件

```bash
tiup cluster start 集群名字
tiup cluster start 集群名字 -R 组件1,组件2
```

启动集群操作会按 PD -> TiKV -> Pump -> TiDB -> TiFlash -> Drainer 的顺序启动整个 TiDB 集群所有组件（同时也会启动监控组件）

## 8、验证

①查看当前集群的状态

```bash
tiup cluster display 集群名字
```

②查看哪个 PD 节点提供了 TiDB Dashboard 服务

```bash
tiup cluster display 集群名字 --dashboard
```



## 9、其他信息

- Grafana默认用户名密码为：`admin/admin`

- 监控Dashboard默认用户名密码为：``root 密码为空``

## 10、集群其他操作

### ①修改集群组件配置参数

集群运行过程中，如果需要调整某个组件的参数，可以使用 `edit-config` 命令来编辑参数。具体的操作步骤如下：

1. 以编辑模式打开该集群的配置文件：

   ```bash
   tiup cluster edit-config ${cluster-name}
   ```

2. 设置参数：

   首先确定配置的生效范围，有以下两种生效范围：

   - 如果配置的生效范围为该组件全局，则配置到 `server_configs`。例如：

     ```text
     server_configs:
       tidb:
         log.slow-threshold: 300
     ```

   - 如果配置的生效范围为某个节点，则配置到具体节点的 `config` 中。例如：

     ```text
     tidb_servers:
     - host: 10.0.1.11
         port: 4000
         config:
             log.slow-threshold: 300
     ```

   参数的格式参考 [TiUP 配置参数模版](https://github.com/pingcap/tiup/blob/master/examples/topology.example.yaml)。**配置项层次结构使用 `.` 表示**。关于组件的更多配置参数说明，可参考 [tidb `config.toml.example`](https://github.com/pingcap/tidb/blob/v4.0.0-rc/config/config.toml.example)、[tikv `config.toml.example`](https://github.com/tikv/tikv/blob/v4.0.0-rc/etc/config-template.toml) 和 [pd `config.toml.example`](https://github.com/pingcap/pd/blob/v4.0.0-rc/conf/config.toml)。

3. 执行 `reload` 命令滚动分发配置、重启相应组件：

   ```bash
   tiup cluster reload ${cluster-name} [-N <nodes>] [-R <roles>]
   ```

**示例**：如果要调整 tidb-server 中事务大小限制参数 `txn-total-size-limit` 为 `1G`，该参数位于 [performance](https://github.com/pingcap/tidb/blob/v4.0.0-rc/config/config.toml.example) 模块下，调整后的配置如下：

```text
server_configs:
  tidb:
    performance.txn-total-size-limit: 1073741824
```

然后执行 `tiup cluster reload ${cluster-name} -R tidb` 命令滚动重启。

### ②重命名集群

部署并启动集群后，可以通过 `tiup cluster rename` 命令来对集群重命名：

```bash
tiup cluster rename ${cluster-name} ${new-name}
```

> **注意：**
>
> - 重命名集群会重启监控（Prometheus 和 Grafana）。
> - 重命名集群之后 Grafana 可能会残留一些旧集群名的面板，需要手动删除这些面板。

### ②关闭集群

关闭集群操作会按 Drainer -> TiFlash -> TiDB -> Pump -> TiKV -> PD 的顺序关闭整个 TiDB 集群所有组件（同时也会关闭监控组件）：

```bash
tiup cluster stop ${cluster-name}
```

和 `start` 命令类似，`stop` 命令也支持通过 `-R` 和 `-N` 参数来只停止部分组件。

例如，下列命令只停止 TiDB 组件：

```bash
tiup cluster stop ${cluster-name} -R tidb
```

下列命令只停止 `1.2.3.4` 和 `1.2.3.5` 这两台机器上的 TiDB 组件：

```bash
tiup cluster stop ${cluster-name} -N 1.2.3.4:4000,1.2.3.5:4000
```

### ④清除集群数据

此操作会关闭所有服务，并清空其数据目录或/和日志目录，并且无法恢复，需要**谨慎操作**。

清空集群所有服务的数据，但保留日志：

```bash
tiup cluster clean ${cluster-name} --data
```

清空集群所有服务的日志，但保留数据：

```bash
tiup cluster clean ${cluster-name} --log
```

清空集群所有服务的数据和日志：

```bash
tiup cluster clean ${cluster-name} --all 
```

清空 Prometheus 以外的所有服务的日志和数据：

```bash
tiup cluster clean ${cluster-name} --all --ignore-role prometheus
```

清空节点 `192.168.1.70:9000` 以外的所有服务的日志和数据：

```bash
tiup cluster clean ${cluster-name} --all --ignore-node 192.168.1.70:9000
```

清空部署在 `172.16.13.12` 以外的所有服务的日志和数据：

```bash
tiup cluster clean ${cluster-name} --all --ignore-node 192.168.1.70
```

### ⑤删除集群

```bash
tiup cluster destroy 集群名字
```

