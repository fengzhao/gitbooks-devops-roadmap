# Apache Pulsar的命令行

# 一、pulsar-admin

pulsar-admin工具，可以用来管理Pulsar的集群、Brokers、名称空间，租户等。

参考文档：https://pulsar.apache.org/docs/en/pulsar-admin/#list-failure-domains

## 1、命令格式

```bash
pulsar-admin 命令 子命令 参数
```



子命令

- `broker-stats`：收集Brokers的统计信息
- `brokers`：操作Brokers
- `clusters`：操作集群
- `functions`：操作Pulsar函数
- `functions-worker`：收集Pulsar函数Brokers的统计信息
- `namespaces`：操作管理命令空间
- `ns-isolation-policy`：操作管理命令空间的隔离策略
- `sources`：
- `sinks`
- `topics`：操作管理主题
- `tenants`：操作管理多租户
- `resource-quotas`：操作管理资源配额
- `schemas`：操作管理主题关联的模式



## 2、常用操作

