# Pulsar

# 一、简介

## 1、功能特性

## 2、Pulsar架构

- **Broker**
    - 一个或多个Broker
    - 处理和负载接受到的生产者发送的消息数据
    - 调度消息发送给消费者
    - 与Zookeeper进行通信以处理各种协调任务，
    - 将消息存储在BookKeeper实例（又称为bookies）中
    - 依赖于ZooKeeper集群来执行某些任务，
    - 等等
- **Apache Zookeeper（Standby/Cluster）**
    - Pulsar使用ZK存储元数据、集群配置，还有协调各Broker
    - 协调由那个Broker响应数据处理
    - 存储Topic主题的元数据
- **Apache BookKeeper（又称为bookies）**
    - 由一个或多个bookies组成的BookKeeper集群来存储需要持久化的消息数据，和消费者消费消息的游标offset
    - Apache BookKeeper是一个分布式的WAL(write-ahead log)系统

# 二、基础概念

{persistent|non-persistent}://tenant/namespace/topic
{持久化|非持久化}://租户/命名空间/主题

topic主题类型
- 持久化的
- 非持久化的

支持的消息压缩格式：
- LZ4
- ZLIB
- ZSTD
- SNAPPY

生产者发送消息模式
- 同步：发送每条消息后，生产者将等待Broker的确认。如果未收到确认，则生产者将发送操作视为失败
- 异步：将把消息放于阻塞队列中，并立即返回。然后，客户端将在后台将消息发送给 broker
       如果队列已满(最大大小可配置)，则调用 API 时，producer 可能会立即被阻止或失败，具体取决于传递给 producer 的参数。

消费者接受消息模式
- 同步
- 异步

订阅模式
- exclusive(独家)：只允许有一个消费者
- shared（共享）：允许多个消费者，消费者间机会均等。消息通过轮询机制分发给不同的消费者，并且每个消息仅会被分发给一个消费者。当消费者断开连接，所有发送给他，但没有被确认的消息将被重新安排，分发给其它存活的消费者。
- failover(灾备)：允许多个消费者，消费者有主从之分，主消费者负责接受数据，主消费者挂掉以后，从消费者代替主消费者接着接受数据
- key_shared：允许多个消费者，具有相同key或相同订阅key的消息仅传递给一个使用者。
              不管消息被重新发送多少次，它都会被发送到同一使用者。当消费者连接或断开连接时，将导致服务的消费者更改某些消息键。

多主题订阅
- 当consumer订阅pulsar的主题时，它默认指定订阅了一个主题，例如：persistent://public/default/my-topic。
- 从Pulsar的1.23.0-incubating的版本开始，Pulsar消费者可以同时订阅多个topic
- 多主题订阅方式：
  - 正则匹配：persistent://public/default/finance-.*
  - 明确指定的topic列表

非持久化主题内的数据处理速度比持久化主题快

消息的默认保留，过期处理方式
- 立即删除消费者已确认的所有消息
- 以消息backlog的形式，持久保存所有的未被确认消息

Pulsar 支持保证一条消息只能在broker服务端被持久化一次的特性，即消息去重功能





# 参考

1. https://jack-vanlightly.com/blog/2018/10/2/understanding-how-apache-pulsar-works

