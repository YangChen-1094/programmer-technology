# 后端技术学习笔记

<div align="center">

**涵盖 MySQL、Redis、消息队列、Kubernetes、分布式系统、算法等��心后端技术**

![MySQL](https://img.shields.io/badge/MySQL-27%20篇-blue)
![Redis](https://img.shields.io/badge/Redis-9%20篇-red)
![Kafka](https://img.shields.io/badge/Kafka-4%20篇-orange)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-5%20篇-orange)
![Pulsar](https://img.shields.io/badge/Pulsar-4%20篇-orange)
![K8S](https://img.shields.io/badge/K8S-10%20篇-blue)
![分布式](https://img.shields.io/badge/分布式-7%20篇-purple)
![算法](https://img.shields.io/badge/算法-1%20篇-green)

</div>

> 本仓库为后端技术学习文档集合，适合后端开发者进阶学习。所有文档均为中文编写，包含丰富的配图和代码示例。

## 📚 目录

- [MySQL](#mysql) - 27篇文档
- [Redis](#redis) - 9篇文档
- [消息队列](#消息队列) - Kafka、RabbitMQ、Pulsar
- [Kubernetes](#kubernetes) - 10篇文档
- [分布式系统](#分布式系统) - CAP、ZooKeeper、分布式事务
- [算法](#算法)

---

## MySQL

### 基础原理
- [SQL语句的执行过程](docs/mysql/1、SQL语句的执行过程.md) - 连接器、查询缓存、分析器、优化器、执行器
- [索引和锁](docs/mysql/2、索引和锁.md)
- [MyISAM和InnoDB的区别](docs/mysql/10、MyISAM和InnoDB的区别.md)

### 事务与并发控制
- [MVCC多版本并发控制](docs/mysql/4、MVCC多版本并发控制.md)
- [事务隔离级别](docs/mysql/4.1、事务隔离级别.md)
- [InnoDB的七种锁](docs/mysql/5、InnoDB的七种锁.md)
- [MySQL排查锁问题](docs/mysql/15、mysql排查锁问题.md)

### 日志与恢复机制
- [UndoLog、RedoLog、BinLog区别](docs/mysql/9、undolog、redolog、binlog区别.md)
- [MySQL刷脏和数据恢复](docs/mysql/20、Mysql刷脏和数据恢复.md)
- [Buffer Pool、Flush链表、Free链表](docs/mysql/22、buffer pool、flush链表、free链表.md)

### 索引优化
- [最左匹配原则](docs/mysql/6、最左匹配原则.md)
- [Explain 索引使用情况](docs/mysql/12、Explain 索引使用情况.md)
- [MySQL选错索引 索引不生效](docs/mysql/13、mysql选错索引 索引不生效.md)
- [MySQL是如何排序的](docs/mysql/14、mysql是如何排序的.md)
- [MySQL常用优化方法](docs/mysql/7、Mysql常用优化方法.md)

### 高级专题
- [MySQL主从复制原理](docs/mysql/9、Mysql-主从复制原理.md)
- [MySQL大表操作](docs/mysql/3、mysql大表操作.md)
- [MySQL连接池](docs/mysql/21、mysql连接池.md)
- [MySQL中count()函数是怎么运行的](docs/mysql/24、mysql中count()函数是怎么运行的.md)
- [LEFT JOIN、RIGHT JOIN和JOIN的区别](docs/mysql/8、left join、right join和join的区别.md)
- [VARCHAR与TEXT的区别](docs/mysql/27、varchar与text的区别.md)

### 数据一致性
- [如何保障MySQL和Redis之间的数据一致性？](docs/mysql/11、如何保障mysql和redis之间的数据一致性？.md)
- [MySQL与Elasticsearch](docs/mysql/26、Mysql与Elasticsearch.md)

---

## Redis

### 高可用与集群
- [Redis高可用，哨兵模式搭建](docs/redis/1、Redis高可用，哨兵模式搭建.md)
- [Redis查看哨兵进程的是如何工作的](docs/redis/2、Redis查看哨兵进程的是如何工作的.md)
- [Redis哨兵模式高可用-故障转移](docs/redis/3、Redis哨兵模式高可用-故障转移.md)
- [Redis cluster集群模式](docs/redis/4、Redis cluster集群模式.md)

### 底层数据结构
- [Redis-string底层实现](docs/redis/5-1、Redis-string底层实现.md)
- [Redis-list底层实现](docs/redis/5-2、Redis-list底层实现.md)
- [Redis-hash底层实现](docs/redis/5-3、Redis-hash底层实现.md)
- [Redis-set底层实现](docs/redis/5-4、Redis-set底层实现.md)
- [Redis-zset底层实现](docs/redis/5-5、Redis-zset底层实现.md)

### 持久化与同步
- [Redis持久化](docs/redis/6、Redis持久化.md) - RDB与AOF
- [Redis主从同步](docs/redis/7、Redis主从同步.md)

### 其他
- [Redis的单线程与多线程](docs/redis/8、redis的单线程与多线程.md)
- [如何保证redis与mysql数据一致性](docs/redis/9、如何保证redis与mysql数据一致性.md)

---

## 消息队列

### Kafka
- [初识Kafka](docs/middleware/1、初识kafka.md)
- [Kafka的生产、存储、消费过程](docs/middleware/1.1、kafka的生产、存储、消费过程.md)
- [ZooKeeper和Kafka的关系](docs/middleware/2、zookeeper和Kafka的关系.md)
- [关于Kafka的问题](docs/middleware/1.2、关于kafka的问题.md)

### RabbitMQ
- [RabbitMQ入门](docs/middleware/4.1、rabbitmq入门.md) - 应用场景（异步、解耦、削峰）
- [RabbitMQ重点详解](docs/middleware/4.2、RabbitMq重点详解.md) - 核心概念、交换机类型、消息可靠性
- [RabbitMQ与Kafka的对比](docs/middleware/4.3、rabbitmq与kafka的对比.md)
- [RabbitMQ集群部署](docs/middleware/5、RabbitMQ集群部署.md)
- [Raft一致性算法](docs/middleware/4、RabbitMq.md) - Leader选举、日志复制

### Pulsar
- [Pulsar入门](docs/middleware/6、Pulsar.md)
- [Pulsar之Broker](docs/middleware/6.1、Pulsar之broker.md)
- [Pulsar之BookKeeper存储](docs/middleware/6.2、Pulsar之bookkeeper存储.md)
- [Pulsar消费者与消息](docs/middleware/6.3、Pulsar消费者与消息.md)

---

## Kubernetes

### 基础架构与组件
- [K8S各个组件](docs/K8S/1、K8S各个组件.md) - ApiServer、Scheduler、ControllerManager、Etcd、kube-proxy、kubelet、Pod
- [controllerManager控制器管理者](docs/K8S/2、controllerManager控制器管理者.md)
- [kube-proxy组件](docs/K8S/5、kube-proxy组件.md)

### 工作负载管理
- [Pod控制器](docs/K8S/3、Pod控制器.md) - ReplicaSet、Deployment、HPA、DaemonSet、Job、Cronjob、StatefulSet
- [deployment、service和endpoint](docs/K8S/4、deployment、service和endpoint.md) - Service类型、负载均衡、服务发现
- [StatefulSets状态服务](docs/K8S/9、StatefulSets状态服务.md) - 有状态应用管理
- [Pod健康监测机制](docs/K8S/10、pod的健康监测机制.md) - livenessProbe、ReadinessProbe、startupProbe

### 服务与网络
- [K8S服务注册与发现](docs/K8S/7、K8S 服务注册与发现.md)

### 实践与工具
- [K8S相关命令操作](docs/K8S/8、K8S相关命令操作.md) - kubectl常用命令
- [Chaos Mesh混沌测试工具](docs/K8S/6、chaos mesh混沌测试工具.md) - 故障注入、混沌工程

---

## 分布式系统

### 基础理论
- [什么是分布式系统](docs/分布式/1、什么是分布式系统.md) - 分布式系统概述、集中式 vs 分布式、集群与分布式的关系
- [CAP模型](docs/分布式/2、CAP模型.md) - 一致性、可用性、分区容错性、CA/CP/AP 模型
- [分布式和微服务的区别](docs/分布式/2、分布式和微服务的区别.md) - 分布式系统与微服务架构的对比

### 协调服务
- [ZooKeeper是什么](docs/分布式/3、zookeeper是什么.md) - ZooKeeper 简介、数据模型、Watch 机制
- [ZooKeeper架构](docs/分布式/4、zookeeper架构.md) - Leader 选举、ZAB 协议、集群架构
- [ZooKeeper伪集群模式、分布式锁](docs/分布式/4、zookeeper伪集群模式、分布式锁（redis、etcd、zookeeper）.md) - Redis/ETCD/ZooKeeper 分布式锁对比

### 分布式事务
- [分布式事务](docs/分布式/5、分布式事务.md) - XA 两阶段提交、三阶段提交、TCC、MQ 事务
- [RocketMQ的分布式事务](docs/分布式/5.1、RocketMQ的分布式事务.md) - 消息事务实现原理

---

## 算法

### 查找算法
- [查找算法](docs/algorithm/Search.md) - 二分查找、插值查找、斐波那契查找、分块查找、树表查找

---

## 📖 文档说明

- **中文编写** - 所有文档均为中文编写，便于理解学习
- **图文并茂** - 包含大量配图说明关键概念和架构
- **代码示例** - 使用 PHP 演示算法原理，通俗易懂
- **系统全面** - 涵盖从基础原理到高级应用的完整知识体系
- **实用导向** - 注重实际应用场景和问题解决方案

## 🎯 学习路径

### 初级 → 中级
1. **MySQL 基础原理** → SQL 执行过程、索引和锁
2. **Redis 基础** → 数据结构、持久化方式
3. **消息队列入门** → Kafka 基本概念、RabbitMQ 应用场景

### 中级 → 高级
1. **MySQL 进阶** → 事务隔离级别、MVCC、主从复制
2. **Redis 高可用** → 哨兵模式、集群模式
3. **消息队列深入** → 消息可靠性、集群部署、Raft 算法

### 高级 → 架构师
1. **性能优化** → 索引优化、慢查询排查、连接池
2. **分布式系统** → CAP 理论、ZooKeeper 协调服务、分布式事务
3. **容器编排** → Kubernetes 架构、服务发现、混沌工程

## 📂 目录结构

```
programmer-technology/
├── docs/
│   ├── algorithm/          # 算法文档
│   ├── mysql/              # MySQL 文档（27篇）
│   ├── redis/              # Redis 文档（9篇）
│   ├── middleware/         # 中间件文档（Kafka/RabbitMQ/Pulsar）
│   ├── K8S/                # Kubernetes 文档（10篇）
│   └── rabbitmq/           # RabbitMQ 独立目录
└── README.md               # 本文件
```
