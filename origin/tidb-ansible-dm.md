# DM(Data Migration)及Ansible部署

# 一、简介

[DM](https://github.com/pingcap/dm) (Data Migration) 是一体化的数据同步任务管理平台，支持从 MySQL 或 MariaDB 到 TiDB 的全量数据迁移和增量数据同步。使用 DM 工具有利于简化错误处理流程，降低运维成本。

## DM 架构

DM 主要包括三个组件：DM-master，DM-worker 和 dmctl。

![](../assets/dm-architecture.png)

### DM-master

DM-master 负责管理和调度数据同步任务的各项操作。

- 保存 DM 集群的拓扑信息
- 监控 DM-worker 进程的运行状态
- 监控数据同步任务的运行状态
- 提供数据同步任务管理的统一入口
- 协调分库分表场景下各个实例分表的 DDL 同步

### DM-worker

DM-worker 负责执行具体的数据同步任务。

- 将 binlog 数据持久化保存在本地
- 保存数据同步子任务的配置信息
- 编排数据同步子任务的运行
- 监控数据同步子任务的运行状态

DM-worker 启动后，会自动同步上游 binlog 至本地配置目录（如果使用 DM-Ansible 部署 DM 集群，默认的同步目录为 `/relay_log`）

### dmctl

dmctl 是用来控制 DM 集群的命令行工具。

- 创建、更新或删除数据同步任务
- 查看数据同步任务状态
- 处理数据同步任务错误
- 校验数据同步任务配置的正确性



# 二、Ansible部署



