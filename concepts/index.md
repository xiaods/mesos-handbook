# 概念原理

本章将对 Mesos 的概念中方方面面进行介绍，以便读者对 Mesos项目能有比较全面的了解，包括：Mesos 历史、生态、能够做什么、怎样搭建一个 Mesos集群以及 Mesos 工作原理

Mesos，一个 Apache 开源项目，致力于构建数据中心级别的集群管理系统。它通过虚拟化物理主机（或者虚拟主机）之上的 CPU、内存、磁盘以及其它计算资源，构建高容错的、可动态伸缩的分布式系统，让运行计算任务快捷有效。Mesos 帮助开发者可以在一个统一的资源池之上运行应用，简化了构建分布式系统的复杂度。为了应对企业应用的复杂需求，开发者可以把各种优秀的计算框架安装到 Mesos 集群中，通过统一的 API 接口来调用管理，从而让弹性扩展的能力在整个数据中心甚至云计算环境中可以被快速的利用起来。Mesos 就像 Linux 中的 Kernel，已经成为构建分布式系统的Kernel。

## 1.1 Mesos 简介

Apache Mesos是由美国伯克利大学(UCB)的AMPLab研发并贡献到 Apache 基金会的一款开源群集管理系统，支持 Hadoop、ElasticSearch、Spark、Storm 和 Kafka 等应用架构。Mesos特性如下：

- 弹性扩展支持10,000个计算节点 
- 使用ZooKeeper 实现 Master 和 Slave 的多容错副本技术
- 支持 Docker 容器技术
- 支持原生的 Linux 容器技术隔离任务
- 基于多资源调度，包括内存，CPU、磁盘、端口
- 提供 Java，Python，C++等多种语言 API接口开发新的分布式应用
- 提供 Web UI 界面查看集群状态

Apache Mesos架构图如下：

![](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

通过以上架构图，我们可以了解到：

- Mesos本身包含两个组件:Master Daemon和Agent Daemon。
  - Master Daemon
    - 管理所有的 Slave Daemon。
    - 用 Resource Offers 实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。
    - 限制和提供资源给应用框架使用。
    - 使用可拔插的模块化的架构，方便增加新的策略控制机制。
  - Agent Daemon
    - 负责接收和管理 Master 发来的需求任务(Task)
    - 支持使用各种环境运行各种任务(Task)，如Docker、VM、进程调度(纯硬件)。
- Mesos上的任务(Task)由2个组件管理:调度器(Scheduler)和执行进程(Executor Process)
  - 调度器(Scheduler)
    - 调度器通过注册 Mesos Master获得集群资源调度权限
    - 调度器( Scheduler)可通过 MesosSchedule Driver 接口和 Mesos Master 交互
  - 执行进程(Executor Process)
    - 用于启动框架内部的任务(Task)
    - 不同的调度器使用不同的执行进程(Executor Process)
- Mesos 集群为了避免单点故障，所以使用 Zookeeper 提供高容错的副本机制。

### 1.1.1 Mesos的运行方式

下图描述了一个 Framework 如何通过调度来运行一个 Task
![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

事件流程:

1. Agent1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Agent1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Agent1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
4. 最后，Master 发送这些 Tasks 给 Agent1。然后，Agent1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

> 注意：当 Tasks 完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。

### 1.1.2 Meoss的应用场景

- **容器编排**

  Mesos本身是一个分布式资源调度管理系统，而且是一个比较开放通用的资源调度管理系统。其北向提供了开放性的Framework框架平台，允许其他应用框架的友好接入（如spark、hadoop等），其南向定义了Executor执行器机制，允许容器、虚拟机等作为执行器进行任务的处理等工作。容器技术随着Docker的出现，目前越来越受到热捧，关于如何更好更快的将容器技术应用到生产实践中全世界都在进行广泛的实践和探索。容器技术不可以质疑的是最佳的执行者，但其缺少一个上层的编排调度系统。应用框架+Mesos+容器的架构当前被大量的讨论，也有一些企业在对这一架构进行了实践，总体上来说是一个比较稳定可靠合理的架构。因为这个架构下，不管对于应用框架还是对于执行器来说都是统一通用的，这个样似乎更加符合DCOS的设想。


- **提升资源利用率**

  提升资源利用率这个词语一定会和虚拟化或云计算一起出现，的确目前在各个层面都在追逐提升资源利用率，因为在大规模服务器的数据中心中，资源利用率的提升代表着更少的资源投入。对于使用超过50台服务器的公司而言，一个通常的使用Mesos的动机就是提升资源利用率，并且减少运维成本。目前已经有许多这样的公司，比如各种公有云和私有云服务的提供商。在Ebay的案例中，它们曾经在Mesos上运行Jenkins这样可以减少虚拟机的使用。Mesosphere也发布了相关的文章对于HubSpot(运行在AWS上)的案例研究,文章中介绍了HubSpot是如何使用几十台大型的服务器来替代了几百台小型的服务器，使得硬件的利用率更高。


- 批处理应用和长时间运行应用

  所谓的批处理应用与长时间运行应用共存，是指可以在一个Mesos集群中混合运行批处理应用以及其他的长时间运行应用，这将对资源利用率的提升起到关键作用，同时这也是mesos一直所追求目标：统一的数据中心操作系统(DCOS)。如在一个Mesos集群中可以运行如MapReduce、Spark等批处理应用，也可以运行如Jenkins等普通应用。这样就不用在一个数据中心划分不同的服务运行区域，进行资源的隔离和精确匹配。

## 1.2 总结

Mesos持有独特地两层资源调度机制和面向二次开发友好的调度框架应用接口，使得Mesos在企业内部自研数据中心操作系统选型中得到广泛的应用。在Mesos社区和企业用户的探索和实践下，Mesos正在向着统一的云数据中心操作系统这一伟大目标前进，我们可以乐观地预见或许不久之后Mesos能够像Openstack技术那样成为企业数据中心环境中必不可少的核心技术组件之一。

