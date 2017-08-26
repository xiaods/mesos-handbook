# 7.1 summary

软件的开发及部署，早已形成了一套标准流程，其中非常重要的组成部分就是持续集成（Continuous integration，简称CI）。

**持续集成(CI)** 是一种软件开发实践，其目的是通过频繁的将代码 merge 到主分支来实现产品的快速迭代。在团队合作开发过程中，持续集成不仅可以帮助团队成员及时发现集成错误，而且也避免了个人的分支工作大幅偏离主分支而导致的代码冲突。使用得当，持续集成会极大的提高软件开发效率并保障软件开发质量。

**Jenkins** 是基于 Java 开发的一种持续集成 (CI) 工具，它提供了一种易于使用的持续集成系统。

**Mesos** 是 Apache 下的一个开源的统一资源管理与调度平台，它被称为是分布式系统的内核。

**Marathon** 是注册到 Apache Mesos 上的管理长时应用(long-running applications)的 Framework，如果把 Mesos 比作数据中心 kernel 的话，那么 Marathon 就是 init 或者 upstart 的 daemon。

本章节旨在探讨如何利用 Jenkins，Apache Mesos 和 Marathon 搭建一套**弹性**的，**高可用**的持续集成环境。