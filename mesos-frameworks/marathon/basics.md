# Marathon 简介

Marathon 是 Mesosphere 专门为 Mesos 开发的长时任务处理框架，并且在 GitHub
上开源，利用 Marathon 以及一些其它开源软件，加上一些本地开发，
结合当前流行的容器解决方案 Docker, 就可以搭建一个基于 Mesos 的
PaaS(Platform as a Service) 平台。

当然，Docker 并不是必须的，但是由于 Docker 能够很好的解决构建、打包、
分发和运行的问题，所以加入 Docker 也就成了不二之选，Mesos, Marathon 都原生支持
Docker，所以使用起来非常方便。

截止本书写作之时，Marathon 的最新版本为 0.13.0，支持以下特性：

  - HA(High Availability)
  - Constraints
  - 服务发现和负载均衡
  - 健康检查
  - 事件机制
  - Web UI
  - RESTful API
  - Basic Auth 以及支持 SSL
  - Metrics

下面将对以上特性逐一介绍。

## HA(High Availability)

Marathon 的 HA 实现非常简单，Marathon 将所有数据持久化到 ZooKeeper 服务中，
自身是一个无状态的服务，而各个服务实例之间通过 ZooKeeper 实现 Leader 选举，
任意时刻，只有最多一个服务实例作为 Leader，其它实例则作为 Follower，
任何到达 Follower 的请求都将被转发到 Leader。所以，任意时刻，只要有一个 Marathon
服务实例存活，整个 Marathon 服务都是可用的。

所以，在生产环境中，要实现 Marathon 服务的高可用性，至少需要搭建 2 个 Marathon
服务实例组成一个服务。

## Constraints

在生产环境中，保持集群中所有结点的一致性显然是不可能的事，总是有一些特性能够将
Mesos 计算结点区分开来，例如：

  - 结点所在的机架
  - 结点的网络带宽
  - 结点是否可以访问外网，通过何种方式
  - 结点所在机房的运营商
  - 结点是物理机还是虚拟机
  - 等等

所以，Marathon 使用 Constraints 特性来支持将任务运行在具有指定特性的计算结点上。

## 服务发现和负载均衡

想象一下在传统的 Web 服务部署中，通常是将服务直接部署在固定的一台或多台物理机或者
虚拟机上，然后，将这些服务器的 IP 地址绑定到负载均衡器中，对外提供服务。但是，
当服务器故障时怎么办呢？需要故障处理和恢复，如果故障的服务器不能恢复，
需要部署新的服务器并加入到负载均衡器中，避免影响服务的稳定性。

想象一下，将这样的 Web 服务通过 Marathon 运行在 Mesos 集群中会是什么样？Marathon
会自动将服务的多个实例运行在一些计算结点上，当结点或者实例故障时，会自动重启一个新的实例，
那么这里存在的问题是在任意时刻怎样知道服务实例运行的地址呢？

服务发现和负载均衡特性就是为了解决这个问题，Marathon 能够为长时任务生成 HAProxy
配置文件，如果使用 HAProxy 作为负载均衡器，那么服务实例的故障是完全不用运维的。
虽然 Marathon 没有提供其它负载均衡器的支持，但是已经不乏这样的开源软件，
例如：Bamboo, Consule 等。

## 健康检查

由于 Marathon 专为长时任务设计，并且支持自动重启故障的任务，任务的故障有很多种，
计算结点导致的故障可以由 Mesos 通知，但是如果是任务内部的异常，Mesos
则无能为力了，例如：Web 服务正在运行，但是已经不能响应服务请求了，
那么有什么机制能够发现并处理这种异常吗？

熟悉 HAProxy, Nginx 的读者应该知道，它们可以通过检测后端服务器的服务接口来自动屏蔽异常的服务器，
Marathon 也实现了类似的机制，叫做健康检查，能够自动检查服务实例并且在发现异常后做出响应，
例如：杀掉异常的服务实例并且重新启动一个新的服务实例。

## 事件机制

事件机制为开发者提供了无限的可能，开发者可以通过注册到 Marathon 的事件总线，
及时知道自己感兴趣的事件，从而做出相应。

例如：通过事件机制，可以监听到服务实例的变化，自动配置负载均衡器。

## Web UI

Marathon 提供了实用的 Web UI，能够方便用户管理长时任务，实现任务的增、删、查、改操作。
下图是一个 Marathon Web UI 截图。

![FIXME: marathon web ui](assets/marathon-web-ui-home.png)

## RESTful API

Marathon 不仅提供了 Web UI，还提供了 RESTful API，对于开发者来说，良好设计的 API
更值得拥有，你可以通过 Marathon API 实现通过 Web UI 能够实现的所有功能，甚至一些
Web UI 不支持的功能。例如：强制重新部署。

## Basic Auth 以及支持 SSL

Marathon 实现了 Basic Auth，Basic Auth 是一种简单的用户名密码认证方式，目前
Marathon 只支持单用户，所有任务都只能属于这个用户，所以本质上，
如果有多个用户在一个 Marathon 服务上创建了任务，那么是有可能误操作的。

在通信安全方面，Marathon 支持 SSL 加密。

## Metrics

通过 Metrics，用户可以很方便的知道 Marathon
的运行情况，例如：任务数量，使用的资源等等。
