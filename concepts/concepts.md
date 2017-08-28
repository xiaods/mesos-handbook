# 设计理念

Mesos，一个 Apache 开源项目，致力于构建数据中心级别的集群管理系统。它通过虚拟化物理主机（或者虚拟主机）之上的 CPU、内存、磁盘以及其它计算资源，构建高容错的、可动态伸缩的分布式系统，让运行计算任务快捷有效。Mesos 帮助开发者可以在一个统一的资源池之上运行应用，简化了构建分布式系统的复杂度。为了应对企业应用的复杂需求，开发者可以把各种优秀的计算框架安装到 Mesos 集群中，通过统一的 API 接口来调用管理，从而让弹性扩展的能力在整个数据中心甚至云计算环境中可以被快速的利用起来。Mesos 就像 Linux 中的 Kernel，已经成为构建分布式系统的Kernel。

## 1.1 Mesos 简介
 Apache Mesos是由美国伯克利大学(UCB)的AMPLab研发并贡献到 Apache 基金会的一款开源群集管理系统，支持 Hadoop、ElasticSearch、Spark、Storm 和 Kafka 等应用架构。Mesos特性如下：

* 弹性扩展支持10,000个计算节点 
* 使用ZooKeeper 实现 Master 和 Slave 的多容错副本技术
* 支持 Docker 容器技术
* 支持原生的 Linux 容器技术隔离任务
* 基于多资源调度，包括内存，CPU、磁盘、端口
* 提供 Java，Python，C++等多种语言 API接口开发新的分布式应用
* 提供 Web UI 界面查看集群状态

Apache Mesos架构图如下：
![](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

 通过以上架构图，我们可以了解到：

* Mesos本身包含两个组件:Master Daemon和Slave Daemon。
    * Master Daemon
        * 管理所有的 Slave Daemon。
        * 用[Resource Offers](https://github.com/Dataman-Cloud/Mesos-CN/blob/master/OverView/Mesos-of-ResourceOffer.md)实现跨应用细粒度资源共享，如 cpu、内存、磁盘、网络等。
        * 限制和提供资源给应用框架使用。
        * 使用可拔插的模块化的架构，方便增加新的策略控制机制。
    * Slave Daemon
        * 负责接收和管理 Master 发来的需求 Task
        * 支持使用各种环境运行各种 Task，如Docker、VM、进程调度(纯硬件)。

* Mesos上的task由2个组件管理:调度器(Scheduler)和执行进程(Executor Process)
    * 调度器(Scheduler)
        * 调度器通过注册 Mesos Master获得集群资源调度权限
        * 调度器可通过 MesosSchedule Driver 接口和 Mesos Master 交互
    * 执行进程(Executor Process)
        * 用于启动框架内部的 Task
        * 不同的调度器使用不同的 Executor

* Mesos 集群为了避免单点故障，所以使用 Zookeeper 提供高容错的副本机制。


### 1.1.1 Mesos的运行方式

下图描述了一个 Framework 如何通过调度来运行一个 Task
![Resource offer](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

事件流程:
1. Slave1 向 Master 报告，有4个CPU和4 GB内存可用
2. Master 发送一个 Resource Offer 给 Framework1 来描述 Slave1 有多少可用资源
3. FrameWork1 中的 FW Scheduler会答复 Master，我有两个 Task 需要运行在 Slave1，一个 Task 需要<2个CPU，1 GB内存>，另外一个Task需要<1个CPU，2 GB内存>
4. 最后，Master 发送这些 Tasks 给 Slave1。然后，Slave1还有1个CPU和1 GB内存没有使用，所以分配模块可以把这些资源提供给 Framework2

> 注意：当 Tasks 完成和有新的空闲资源时，Resource Offer会不断重复这一个过程。


### 1.1.2 Mesos与虚拟化、容器技术对比


### 1.1.3 Meoss的应用场景

* 容器编排


Mesos本身是一个分布式资源调度管理系统，而且是一个比较开放通用的资源调度管理系统。其北向提供了开放性的Framework框架平台，允许其他应用框架的友好接入（如spark、hadoop等），其南向定义了Executor执行器机制，允许容器、虚拟机等作为执行器进行任务的处理等工作。容器技术随着Docker的出现，目前越来越受到热捧，关于如何更好更快的将容器技术应用到生产实践中全世界都在进行广泛的实践和探索。容器技术不可以质疑的是最佳的执行者，但其缺少一个上层的编排调度系统。应用框架+Mesos+容器的架构当前被大量的讨论，也有一些企业在对这一架构进行了实践，总体上来说是一个比较稳定可靠合理的架构。因为这个架构下，不管对于应用框架还是对于执行器来说都是统一通用的，这个样似乎更加符合DCOS的设想。


* 提升资源利用率

提升资源利用率这个词语一定会和虚拟化或云计算一起出现，的确目前在各个层面都在追逐提升资源利用率，因为在大规模服务器的数据中心中，资源利用率的提升代表着更少的资源投入。对于使用超过50台服务器的公司而言，一个通常的使用Mesos的动机就是提升资源利用率，并且减少运维成本。目前已经有许多这样的公司，比如各种公有云和私有云服务的提供商。在Ebay的案例中，它们曾经在Mesos上运行Jenkins这样可以减少虚拟机的使用。Mesosphere也发布了相关的文章对于HubSpot(运行在AWS上)的案例研究,文章中介绍了HubSpot是如何使用几十台大型的服务器来替代了几百台小型的服务器，使得硬件的利用率更高。


* 批处理服务和普通处理服务共存

所谓的批处理服务与普通服务共存可以在一个共享的Mesos集群中同时运行批处理任务以及其他的普通服务，这将对资源利用率的提升起到关键作用，同时这也是mesos一直所追求目标：统一的DCOS。如在一个Mesos集群中可以运行如M&R、Spark等批处理服务也可以运行如jenkins等普通服务。这样就不用在一个数据中心划分不同的服务运行区域，进行资源的隔离和精确匹配。



## 1.2 Meos安装指南


### 1.2.1 Mesos集群组件介绍





### 1.2.2 Mesos生产环境配置介绍



## 1.3 安装Mesos和Zookeeper

### 1.3.1 标准包安装





### 1.3.2 源码包安装


## 1.4 Docker安装配置指南




### 1.4.1 安装方法




### 1.4.2 配置方法




### 1.4.3 配置Mesos Slave与Docker




## 1.5 Mesos升级




### 1.5.1 升级流程
•	 按照官所升级版的要求编译和安装所有依赖模块，以保证更新后的版本不存在依赖包的问题。

•	在Msster节点安装新版本的源码包，并重启master节点。

•	在slave节点安装新版本的源码包，并重启slave节点。

•	通过连接本地库/jar等来更新调度器

•	重启调度器.

•	如果有必要也可以通过连接本地库/jar来更新执行器（如docker的版本）.





### 1.5.2注意事项

**0.24.x 升级到 0.25.x**

注意：在0.25.x版本中一些配置文件不需要包含json后缀，不过在0.25版本配置文件是否包含json后缀同样是有效的，包含json后缀的配置文件形式会在后续的版本中逐步的被取消。

Master节点:

•	/state.json 变成 /state

•	/tasks.json 变成/tasks

在slave节点:

•	/state.json 变成 /state

•	/monitor/statistics.json 变成 /monitor/statistics

master和slave节点有变化的配置文件:

•	/files/browse.json 变成 /files/browse

•	/files/debug.json 变成 /files/debug

•	/files/download.json 变成 /files/download

•	/files/read.json 变成 /files/read

注意：在0.25.x版本中，C++，Java,Python的调度包也已经进行了更新，特别地，这些调度包的驱动可以通过生成一个suppressOffers()去直接停止receiving offers进程。

**0.23.x 升级到0.24.x**

注意：在0.24.x版本，master节点在zookeeper中发布信息是通过JSON文件来进行，不在通过protobuf。

**0.22.x 升级到0.23.x**

注意：

•	在master和slave节点上，配置文件stats.json已经被更新成metrics/snapshot。

•	配置文件/master/shutdown已经被弃用

•	为了在decorator模块可以移动元数据（环境变量或者标签），在0.24.x版本中改变了一些decoratorhooks返回值的意义，详细请见更新文档。

•	Slave节点的ping超时时间现在可以在master节点进行配置，可以通过--slave_ping_timeout 和 --max_slave_ping_timeouts进行配置。

•	在0.23.x版本中新增了一个调度driverAPI：acceptOffers，这是launchTasks API的更完善版本。这个driverAPI的作用是允许调度器去接受一个提议并指定一个应用运行列表去进行资源的调度。目前支持的应用包括：LAUNCH (launching tasks), RESERVE (making dynamic reservations), UNRESERVE (releasing dynamic reservations), CREATE (creating persistent volumes) and DESTROY。

•	protobuf源已经扩展成可以包含更多的metadata以便支持存储持久化、动态伸缩、资源超售。这样两个资源对象拥有不同的metadata时，你就不用必须把他们合并。

**0.21.x 升级到0.22.x**

•	在这个版本中，slave检查点标签已经被移除，因为所有的slave节点都会启用这一功能，但是在Frameworks在利用checkpoint登记他们自己的任务时，还需要开启checkpointing.

•	在master和slave节点上，stats.json已经被弃用，需要使用metrics/snapshot。

•	C++/Java/Python调度包已经被更新，尤其是调度driver里包含了一个附加参数可以指定是否去使用模糊的驱动确认。

•	验证API为支持第三方的验证机制在这个版本中有了一个轻微的变化AuthenticationStartMessage.data中的变量类型从string类型变成bytes类型并没有对C++或者over-the-wire表示法造成影响，所以这个一轻微的变化只影响了如jave Python等这些语言包，因为在这些语言中UTF-8 sting和byte数组使用的是不同的类型。

•	所有的mesos参数可以使用file://将他们从文件中读取出来进行传递。包括白名单证书、所有JSON返回参数标签，尽管支持只传送一个绝对路径而不是一个file://，但是这种方式已经被弃用了，如果继续使用就会产生警告信息。

**升级 0.20.x 到 0.21.x**

   关闭slave节点的检验信息已经被弃用，slave节点的检查点标签也已经被弃用，并在下一个版本中会被移除

## 1.6 总结
Mesos具备当前最流行的两层资源调度机制和最开放最稳定的部署框架，mesos社区也是当前最火热的社区之一，目前已经迭代了27个版本，即0.27版本已经发布。在社区和企业的全力探索和实践下，mesos正在向着统一的云数据中心操作系统这一伟大目标前进，我们可以乐观的预想或许不久之后mesos能够像openstack那样被全世界全面的接受，成为每个数据中心所必需的优秀资源调度系统的代名词。













