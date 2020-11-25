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

### ②所有节点关闭系统 swap、关闭透明大页

```bash
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a
sysctl -p
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
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
  arch: "amd64"
  resource_control:
    memory_limit: "6G"
    cpu_quota: "200%"

server_configs:
  tidb:
    oom-action: "cancel"
    mem-quota-query: 25769803776
    log.query-log-max-len: 4096
    log.file.log-rotate: true
    log.file.max-size: 300
    log.file.max-days: 7
    log.file.max-backups: 14
    log.slow-threshold: 300
    binlog.enable: true
    binlog.ignore-error: true
  tikv:
    global.log-rotation-timespan: "168h"
    readpool.unified.max-thread-count: 12
    readpool.storage.use-unified-pool: false
    readpool.coprocessor.use-unified-pool: true
    coprocessor.split-region-on-table: false
    storage.block-cache.capacity: "16GB"
    raftstore.capacity: "10GB"
  pd:
    log.file.max-size: 300
    log.file.max-days: 28
    log.file.max-backups: 14
    log.file.log-rotate: true
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
    ssh_port: 22
    name: pd-1
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/tidb/pd-2379
    data_dir: /data/tidb/pd-2379/data
    log_dir: /data/tidb/pd-2379/logs
  - host: 192.168.1.72
    ssh_port: 22
    name: pd-2
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/tidb/pd-2379
    data_dir: /data/tidb/pd-2379/data
    log_dir: /data/tidb/pd-2379/logs
  - host: 192.168.1.73
    ssh_port: 22
    name: pd-3
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/tidb/pd-2379
    data_dir: /data/tidb/pd-2379/data
    log_dir: /data/tidb/pd-2379/logs

tidb_servers:
  - host: 192.168.1.71
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/tidb-4000"
    log_dir: "/data/tidb/tidb-4000/logs"
    numa_node: "0"
  - host: 192.168.1.72
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/tidb-4000"
    log_dir: "/data/tidb/tidb-4000/logs"
    numa_node: "0"
  - host: 192.168.1.73
    port: 4000
    status_port: 10080
    deploy_dir: "/data/tidb/tidb-4000"
    log_dir: "/data/tidb/tidb-4000/logs"
    numa_node: "0"

tikv_servers:
  - host: 192.168.1.71
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/tikv-20160"
    data_dir: "/data/tidb/tikv-20160/data"
    log_dir: "/data/tidb/tikv-20160/logs"
    numa_node: "0"
    config:
      server.labels: { host: "tikv1" }
  - host: 192.168.1.72
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/tikv-20160"
    data_dir: "/data/tidb/tikv-20160/data"
    log_dir: "/data/tidb/tikv-20160/logs"
    numa_node: "0"
    config:
      server.labels: { host: "tikv2" }
  - host: 192.168.1.73
    port: 20160
    status_port: 20180
    deploy_dir: "/data/tidb/tikv-20160"
    data_dir: "/data/tidb/tikv-20160/data"
    log_dir: "/data/tidb/tikv-20160/logs"
    numa_node: "0"
    config:
      server.labels: { host: "tikv3" }
tiflash_servers:
  - host: 192.168.1.70
    data_dir: "/data/tidb/tiflash-9000/data"
    deploy_dir: "/data/tidb/tiflash-9000"
    log_dir: "/data/tidb/tiflash-9000/logs"
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
    deploy_dir: "/data/tidb/pump-8249"
    data_dir: "/data/tidb/pump-8249/data"
    log_dir: "/data/tidb/pump-8249/logs"
    numa_node: "0"
    # The following configs are used to overwrite the `server_configs.drainer` values.
    config:
      gc: 7
tispark_masters:
  - host: 192.168.1.70
    ssh_port: 22
    port: 7077
    web_port: 8080
    deploy_dir: "/data/tidb/tispark-master-7077"
    java_home: "/opt/jdk"
    spark_config:
      spark.driver.memory: "2g"
      spark.eventLog.enabled: "True"
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
    deploy_dir: "/data/tidb/tispark-worker-7078"
    java_home: "/opt/jdk"

cdc_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 8300
    deploy_dir: "/data/tidb/cdc-8300"
    log_dir: "/data/tidb/cdc-8300/logs"

monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
  deploy_dir: "/data/tidb/monitored-9100"
  data_dir: "/data/tidb/monitored-9100/data"
  log_dir: "/data/tidb/monitored-9100/logs"

monitoring_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 9090
    deploy_dir: "/data/tidb/prometheus-9090"
    data_dir: "/data/tidb/prometheus-9090/data"
    log_dir: "/data/tidb/prometheus-9090/logs"

grafana_servers:
  - host: 192.168.1.70
    ssh_port: 22
    port: 3000
    deploy_dir: "/data/tidb/grafana-3000"

# alertmanager_servers:
#   - host: 192.168.1.70
#     ssh_port: 22
#     web_port: 9093
#     cluster_port: 9094
#     deploy_dir: "/data/tidb/alertmanager-9093"
#     data_dir: "/data/tidb/alertmanager-9093/data"
#     log_dir: "/data/tidb/alertmanager-9093/logs"
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

### ①Grafana默认用户名密码

`admin/admin`

### ②监控Dashboard默认用户名密码

`root 密码为空`

### ③组件拓扑配置文件样例

- 全配置参数模版：https://github.com/pingcap/tiup/blob/master/examples/topology.example.yaml

- TiDB：https://github.com/pingcap/tidb/blob/master/config/config.toml.example
- PD：https://github.com/tikv/pd/blob/v4.0.0-rc/conf/config.toml
- TiKV：https://github.com/tikv/tikv/blob/v4.0.0-rc/etc/config-template.toml

# 四、集群其他操作

## 1、修改集群组件配置参数

集群运行过程中，如果需要调整某个组件的参数，可以使用 `edit-config` 命令来编辑参数。具体的操作步骤如下：

1. 以编辑模式打开该集群的配置文件：

   ```bash
   tiup cluster edit-config ${cluster-name}
   ```

2. 设置参数：

   首先确定配置的生效范围，有以下两种生效范围：

   - 如果配置的生效范围为该组件全局，则配置到 `server_configs`。例如：

     ```yaml
     server_configs:
       tidb:
         log.slow-threshold: 300
     ```

   - 如果配置的生效范围为某个节点，则配置到具体节点的 `config` 中。例如：

     ```yaml
     tidb_servers:
     - host: 10.0.1.11
         port: 4000
         config:
             log.slow-threshold: 300
     ```

   参数的格式参考 [TiUP 配置参数模版](https://github.com/pingcap/tiup/blob/master/examples/topology.example.yaml)。**配置项层次结构使用 `.` 表示**。

3. 执行 `reload` 命令滚动分发配置、重启相应组件：

   ```bash
   tiup cluster reload ${cluster-name} [-N <nodes>] [-R <roles>]
   ```

**示例**：如果要调整 tidb-server 中事务大小限制参数 `txn-total-size-limit` 为 `1G`，该参数位于 [performance](https://github.com/pingcap/tidb/blob/v4.0.0-rc/config/config.toml.example) 模块下，调整后的配置如下：

```yaml
server_configs:
  tidb:
    performance.txn-total-size-limit: 1073741824
```

然后执行 `tiup cluster reload ${cluster-name} -R tidb` 命令滚动重启。

## 2、重命名集群

部署并启动集群后，可以通过 `tiup cluster rename` 命令来对集群重命名：

```bash
tiup cluster rename ${cluster-name} ${new-name}
```

> **注意：**
>
> - 重命名集群会重启监控（Prometheus 和 Grafana）。
> - 重命名集群之后 Grafana 可能会残留一些旧集群名的面板，需要手动删除这些面板。

## 3、关闭集群

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

## 4、清除集群数据

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

## 5、删除集群

```bash
tiup cluster destroy 集群名字
```

## 6、集群组件升级

在官方组件提供了新版之后，你可以使用 `tiup update` 命令来升级组件。除了以下几个参数，该命令的用法基本和 `tiup install` 相同：

- `--all`：升级所有组件
- `--nightly`：升级至 nightly 版本
- `--self`：升级 TiUP 自己至最新版本
- `--force`：强制升级至最新版本

示例一：升级所有组件至最新版本

```shell
tiup update --all
```

示例二：升级所有组件至 nightly 版本

```shell
tiup update --all --nightly
```

示例三：升级 TiUP 至最新版本

```shell
tiup update --self
```

## 7、集群组件扩容

编辑组件节点扩容节点信息的文件 `tidb-servers-scale-out-new-node.yaml`

TIDB配置文件参考

```ini
tidb_servers:
  - host: 10.0.1.5
    ssh_port: 22
    port: 4000
    status_port: 10080
    deploy_dir: /data/deploy/install/deploy/tidb-4000
    log_dir: /data/deploy/install/log/tidb-4000
```

TiKV 配置文件参考：

```ini
tikv_servers:
  - host: 10.0.1.5
    ssh_port: 22
    port: 20160
    status_port: 20180
    deploy_dir: /data/deploy/install/deploy/tikv-20160
    data_dir: /data/deploy/install/data/tikv-20160
    log_dir: /data/deploy/install/log/tikv-20160
```

PD 配置文件参考：

```ini
pd_servers:
  - host: 10.0.1.5
    ssh_port: 22
    name: pd-1
    client_port: 2379
    peer_port: 2380
    deploy_dir: /data/deploy/install/deploy/pd-2379
    data_dir: /data/deploy/install/data/pd-2379
    log_dir: /data/deploy/install/log/pd-2379
```

可以使用 `tiup cluster edit-config <cluster-name>` 查看当前集群的配置信息，因为其中的 `global` 和 `server_configs` 参数配置默认会被 `scale-out.yaml` 继承，因此也会在 `scale-out.yaml` 中生效。

> 此处假设当前执行命令的用户和新增的机器打通了互信，如果不满足已打通互信的条件，需要通过 `-p` 来输入新机器的密码，或通过 `-i` 指定私钥文件。

执行扩容命令

```shell
tiup cluster scale-out 集群名 tidb-servers-scale-out-new-node.yaml
```

预期输出 Scaled cluster `<cluster-name>` out successfully 信息，表示扩容操作成功。

## 8、集群组件实例清理

你可以使用 `tiup clean` 命令来清理组件实例，并删除工作目录。如果在清理之前实例还在运行，会先 kill 相关进程。该命令用法如下：

```bash
tiup clean [tag] [flags]
```

支持以下参数：

- `--all`：清除所有的实例信息

其中 tag 表示要清理的实例 tag，如果使用了 `--all` 则不传递 tag。

示例一：清理 tag 名称为 `experiment` 的组件实例

```shell
tiup clean experiment
```

示例二：清理所有组件实例

```shell
tiup clean --all
```

## 9、集群组件卸载

TiUP 安装的组件会占用本地磁盘空间，如果不想保留过多老版本的组件，可以先查看当前安装了哪些版本的组件，然后再卸载某个组件。

你可以使用 `tiup uninstall` 命令来卸载某个组件的所有版本或者特定版本，也支持卸载所有组件。该命令用法如下：

```bash
tiup uninstall [component][:version] [flags]
```

支持的参数：

- `--all`：卸载所有的组件或版本
- `--self`：卸载 TiUP 自身

component 为要卸载的组件名称，version 为要卸载的版本，这两个都可以省略，省略任何一个都需要加上 `--all` 参数：

- 若省略版本，加 `--all` 表示卸载该组件所有版本
- 若版本和组件都省略，则加 `--all` 表示卸载所有组件及其所有版本

示例一：卸载 v3.0.8 版本的 TiDB

```shell
tiup uninstall tidb:v3.0.8
```

示例二：卸载所有版本的 TiKV

```shell
tiup uninstall tikv --all
```

示例三：卸载所有已经安装的组件

```shell
tiup uninstall --all
```

