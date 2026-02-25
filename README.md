# 后端技术学习笔记

> 本仓库为后端技术学习文档集合，涵盖 MySQL、Redis、消息队列、算法等核心知识点。

## 📚 目录

- [MySQL](#mysql) - 27篇文档
- [Redis](#redis) - 9篇文档
- [消息队列](#消息队列) - Kafka、RabbitMQ、Pulsar
- [算法](#算法)

---

## MySQL

### 基础原理
- [SQL语句的执行过程](docs/docs/mysql/1、SQL语句的执行过程.md) - 连接器、查询缓存、分析器、优化器、执行器
- [索引和锁](docs/docs/mysql/2、索引和锁.md)
- [MyISAM和InnoDB的区别](docs/docs/mysql/10、MyISAM和InnoDB的区别.md)

### 事务与并发控制
- [MVCC多版本并发控制](docs/docs/mysql/4、MVCC多版本并发控制.md)
- [事务隔离级别](docs/docs/mysql/4.1、事务隔离级别.md)
- [InnoDB的七种锁](docs/docs/mysql/5、InnoDB的七种锁.md)
- [MySQL排查锁问题](docs/docs/mysql/15、mysql排查锁问题.md)

### 日志与恢复机制
- [UndoLog、RedoLog、BinLog区别](docs/docs/mysql/9、undolog、redolog、binlog区别.md)
- [MySQL刷脏和数据恢复](docs/docs/mysql/20、Mysql刷脏和数据恢复.md)
- [Buffer Pool、Flush链表、Free链表](docs/docs/mysql/22、buffer pool、flush链表、free链表.md)

### 索引优化
- [最左匹配原则](docs/docs/mysql/6、最左匹配原则.md)
- [Explain 索引使用情况](docs/docs/mysql/12、Explain 索引使用情况.md)
- [MySQL选错索引 索引不生效](docs/docs/mysql/13、mysql选错索引 索引不生效.md)
- [MySQL是如何排序的](docs/docs/mysql/14、mysql是如何排序的.md)
- [MySQL常用优化方法](docs/docs/mysql/7、Mysql常用优化方法.md)

### 高级专题
- [MySQL主从复制原理](docs/docs/mysql/9、Mysql-主从复制原理.md)
- [MySQL大表操作](docs/docs/mysql/3、mysql大表操作.md)
- [MySQL连接池](docs/docs/mysql/21、mysql连接池.md)
- [MySQL中count()函数是怎么运行的](docs/docs/mysql/24、mysql中count()函数是怎么运行的.md)
- [LEFT JOIN、RIGHT JOIN和JOIN的区别](docs/docs/mysql/8、left join、right join和join的区别.md)
- [VARCHAR与TEXT的区别](docs/docs/mysql/27、varchar与text的区别.md)

### 数据一致性
- [如何保障MySQL和Redis之间的数据一致性？](docs/docs/mysql/11、如何保障mysql和redis之间的数据一致性？.md)
- [MySQL与Elasticsearch](docs/docs/mysql/26、Mysql与Elasticsearch.md)

---

## Redis

### 高可用与集群
- [Redis高可用，哨兵模式搭建](docs/docs/redis/1、Redis高可用，哨兵模式搭建.md)
- [Redis查看哨兵进程的是如何工作的](docs/docs/redis/2、Redis查看哨兵进程的是如何工作的.md)
- [Redis哨兵模式高可用-故障转移](docs/docs/redis/3、Redis哨兵模式高可用-故障转移.md)
- [Redis cluster集群模式](docs/docs/redis/4、Redis cluster集群模式.md)

### 底层数据结构
- [Redis-string底层实现](docs/docs/redis/5-1、Redis-string底层实现.md)
- [Redis-list底层实现](docs/docs/redis/5-2、Redis-list底层实现.md)
- [Redis-hash底层实现](docs/docs/redis/5-3、Redis-hash底层实现.md)
- [Redis-set底层实现](docs/docs/redis/5-4、Redis-set底层实现.md)
- [Redis-zset底层实现](docs/docs/redis/5-5、Redis-zset底层实现.md)

### 持久化与同步
- [Redis持久化](docs/docs/redis/6、Redis持久化.md) - RDB与AOF
- [Redis主从同步](docs/docs/redis/7、Redis主从同步.md)

### 其他
- [Redis的单线程与多线程](docs/docs/redis/8、redis的单线程与多线程.md)
- [如何保证redis与mysql数据一致性](docs/docs/redis/9、如何保证redis与mysql数据一致性.md)

---

## 消息队列

### Kafka
- [初识Kafka](docs/docs/middleware/1、初识kafka.md)
- [Kafka的生产、存储、消费过程](docs/docs/middleware/1.1、kafka的生产、存储、消费过程.md)
- [ZooKeeper和Kafka的关系](docs/docs/middleware/2、zookeeper和Kafka的关系.md)
- [关于Kafka的问题](docs/docs/middleware/1.2、关于kafka的问题.md)

### RabbitMQ
- [RabbitMQ入门](docs/docs/middleware/4.1、rabbitmq入门.md) - 应用场景（异步、解耦、削峰）
- [RabbitMQ重点详解](docs/docs/middleware/4.2、RabbitMq重点详解.md) - 核心概念、交换机类型、消息可靠性
- [RabbitMQ与Kafka的对比](docs/docs/middleware/4.3、rabbitmq与kafka的对比.md)
- [RabbitMQ集群部署](docs/docs/middleware/5、RabbitMQ集群部署.md)
- [Raft一致性算法](docs/docs/middleware/4、RabbitMq.md) - Leader选举、日志复制

### Pulsar
- [Pulsar入门](docs/docs/middleware/6、Pulsar.md)
- [Pulsar之Broker](docs/docs/middleware/6.1、Pulsar之broker.md)
- [Pulsar之BookKeeper存储](docs/docs/middleware/6.2、Pulsar之bookkeeper存储.md)
- [Pulsar消费者与消息](docs/docs/middleware/6.3、Pulsar消费者与消息.md)

---

## 算法

### 查找算法
- [查找算法](docs/docs/algorithm/Search.md) - 二分查找、插值查找、斐波那契查找、分块查找、树表查找

---

## 📖 文档说明

- 所有文档均为中文编写
- 代码示例使用 PHP 演示算法原理
- 包含配图说明关键概念
- 适合后端开发者进阶学习

## 📂 目录结构

```
programmer-technology/
├── docs/
│   ├── algorithm/          # 算法文档
│   ├── mysql/              # MySQL 文档（27篇）
│   ├── redis/              # Redis 文档（9篇）
│   ├── middleware/         # 中间件文档（Kafka/RabbitMQ/Pulsar）
│   └── rabbitmq/           # RabbitMQ 独立目录
├── images/                 # 文档配图
├── .claude/                # Claude Code 技能配置
│   └── skills/             # CRUD生成器、测试生成器、代码审查
├── CLAUDE.md               # Claude Code 项目指南
└── README.md               # 本文件
```
