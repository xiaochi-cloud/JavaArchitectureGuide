# Java 架构指南 🚀

欢迎来到 Java 架构指南仓库！这个仓库旨在提供有关 Java 软件架构的见解、最佳实践和学习资源。

## 内容目录

- [设计模式文档](articles/design_patterns/)
- [Redis 文档](articles/redis/)
- [MySQL 文档](articles/mysql/)
- [贡献](CONTRIBUTING.md)
- [许可证](LICENSE)

## 关于 ℹ️

在这个仓库中，您将找到一系列精选的文章、教程、代码示例和资源，涵盖了与 Java 软件架构相关的各种主题。无论您是初学者想要了解基本架构概念，还是经验丰富的开发人员寻求高级技术和最佳实践，本指南旨在提供有价值的见解和实用知识。

## 入门指南 📚

要开始使用本指南，请浏览仓库的内容。您可以浏览目录以找到涵盖各种主题的文章、教程和代码示例，例如：

- 基础
  - 设计模式 [设计模式文档](articles/design_patterns/) 【Done 2024/04/11】
    - [001 七大软件设计原则](articles/design_patterns/001%20七大软件设计原则.md)
    - [002 中介者模式](articles/design_patterns/002%20中介者模式.md)
    - [003 享元模式](articles/design_patterns/003%20享元模式.md)
    - [004 单例模式](articles/design_patterns/004%20单例模式.md)
    - [005 观察者模式](articles/design_patterns/005%20观察者模式.md)
    - [006 迭代器模式](articles/design_patterns/006%20迭代器模式.md)
    - [007 策略模式](articles/design_patterns/007%20策略模式.md)
    - [008 状态模式](articles/design_patterns/008%20状态模式.md)
    - [009 备忘录模式](articles/design_patterns/009%20备忘录模式.md)
    - [010 访问者模式](articles/design_patterns/010%20访问者模式.md)
    - [011 解释器模式](articles/design_patterns/011%20解释器模式.md)
    - [012 建造者模式](articles/design_patterns/012%20建造者模式.md)
    - [013 原型模式](articles/design_patterns/013%20原型模式.md)
    - [014 代理模式](articles/design_patterns/014%20代理模式.md)
    - [015 适配器模式](articles/design_patterns/015%20适配器模式.md)
    - [016 组合模式](articles/design_patterns/016%20组合模式.md)
    - [017 装饰者模式](articles/design_patterns/017%20装饰者模式.md)
    - [018 外观模式](articles/design_patterns/018%20外观模式.md)
    - [019 享元模式](articles/design_patterns/019%20享元模式.md)
    - [020 外观模式](articles/design_patterns/020%20外观模式.md)
    - [021 门面模式](articles/design_patterns/021%20门面模式.md)
    - [022 享元模式](articles/design_patterns/022%20享元模式.md)
    - [999 其他](articles/design_patterns/999%20其他.md)
    - [README](articles/design_patterns/README.md)
  - 算法
  - Java 基础 【TODO】
  - 并发编程 【TODO】
  - Spring 【TODO】
  - Spring Boot 【TODO】
  - MyBatis【TODO】
- 分布式
  - 分布式缓存
    - Redis [Redis 文档](articles/redis/) 浏览关于 Redis 的详细文档，包括基础理论、数据类型、高可用集群、性能优化、安全监控等内容。【Done 2024/04/12】
      - [001 背景&基础理论](articles/redis/001%20背景&基础理论.md)
      - [002 简介&特性](articles/redis/002%20简介&特性.md)
      - [003 数据类型&使用场景](articles/redis/003%20数据类型&使用场景.md)
      - [004 高可用集群](articles/redis/004%20高可用集群.md)
      - [005 常见性能优化方式](articles/redis/005%20常见性能优化方式.md)
      - [006 灾备方案](articles/redis/006%20灾备方案.md)
      - [007 安全&监控](articles/redis/007%20安全&监控.md)
      - [008 常见Java客户端](articles/redis/008%20常见Java客户端.md)
      - [009 案例实践&进阶](articles/redis/009%20案例实践&进阶.md)
      - [010 书籍参考](articles/redis/010%20书籍参考.md)
      - [README](articles/redis/README.md)
  - 消息队列
    - RocketMQ
    - RabbitMQ
    - Kafka
  - 分库分表
    - MyCat
    - Sharding-JDBC\ShardingSphere
  - 任务调度
    - Spring-Scheduler
    - Quartz
    - xxl-Job
  - 通信框架
    - Netty
  - 服务监控
- 微服务 【TODO】
  - Dubbo
  - Spring Cloud Alibaba
    - 注册中心 Nacos
    - 配置中心 Nacos
    - 负载均衡 Ribbon
    - 远程调用 Feign
    - 服务限流 Sentinel
    - 服务网关 GateWay
    - 分布式事务 Seata
- 性能优化
  - Tomcat
  - JVM
  - MySQL [MySQL 文档](articles/mysql/) 发现有关 MySQL 的详细文档，包括数据库基础知识、性能优化、事务处理等方面的内容。 【Doing】
    - [001 InnoDB 内存结构](articles/mysql/001%20InnoDB%20内存结构.md)
    - [002 InnoDB 磁盘结构](articles/mysql/002%20InnoDB%20磁盘结构.md)
    - [003 InnoDB 线程模型](articles/mysql/003%20InnoDB%20线程模型.md)
    - [004 InnoDB 数据文件](articles/mysql/004%20InnoDB%20数据文件.md)
    - [005 InnoDB 参数优化](articles/mysql/005%20InnoDB%20参数优化.md)
- 以及更多！

## 更新日志
- 2024/04/11 :feature：【设计模式】补充设计模式的初始化内容，包括 七大软件设计原则、中介模式、享元模式、单例模式等等 [设计模式概览](articles/design_patterns/README.md)
- 2024/04/12 :feature: 
  - 【Redis】 补充 Redis 的背景、简介、数据类型使用场景、高可用集群、常用性能优化方式 [Redis文档概览](articles/redis/001%20背景&基础理论.md)
  - 【MySQL】初始化 MySQL 的内存结构文档，重点介绍 InnoDB 内存结构 [001 InnoDB 内存结构](articles/mysql/001%20InnoDB%20内存结构.md)
- 2024/04/15 :feature:
  - 【MySQL】 补充 MySQL 磁盘结构 ，包括表空间、数据字典、双写缓冲区、RedoLog、Undo Log 、binLog 以及新版本在磁盘结构上的变化 [002 InnoDB 磁盘结构](articles/mysql/002%20InnoDB%20磁盘结构.md)
- 2024/04/16 :feature:
  - 【MySQL】 补充 InnoDB 线程模型，包括 IO 线程、purge 线程、Page Clean 线程 、Master 线程 [003 InnoDB 线程模型](articles/mysql/003%20InnoDB%20线程模型.md)
  - 【MySQL】 补充 InnoDB 数据文件，包括表空间文件结构、Page 结构、行结构，包括行结构的压缩和存储方式 [004 InnoDB 数据文件](articles/mysql/004%20InnoDB%20数据文件.md)
  - 【MySQL】 补充 InnoDB 参数优化技巧，包括 Buffer Pool 多实例、Buffer Pool 大小设置、缓存性能的评估,日志参数的优化大小的设置、IO线程的优化和设置 [005 InnoDB 参数优化](articles/mysql/005%20InnoDB%20参数优化.md)

## 贡献 🤝

我们鼓励对这个仓库进行贡献！如果您有任何与 Java 架构相关的见解、教程、代码示例或资源，欢迎提交拉取请求。您的贡献将有助于改进这个指南，使其对其他人更有价值。

有关如何贡献的更多信息，请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 文件。

## 许可证 📄

本仓库基于 Apache 许可证 进行许可。

如果您觉得这个仓库有帮助，请给它点个星星，并且不要忘记通过添加自己的见解或建议改进来做出贡献。
