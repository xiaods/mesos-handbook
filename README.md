# Mesos Handbook

## 前言

肖德时篇

Apache Mesos是Apache软件基金会下属的顶级开源项目，它是目前开源分布式集群领域少数应用在生产环境的基础设施软件，通常被用户类比于Linux操作系统的中心系统-Kernel，在全球知名的互联网公司Twitter、AirBnb、Apple、Netflix等公司内部支撑着众多核心的业务系统。随着近几年云计算技术的发展，大量的创业公司也把Mesos系统选择为下一代云平台的核心组件，为搭建分布式系平台系统构建强有力的底座支撑。

笔者是在加入创业公司之后才开始接触Apache Mesos系统的。初次使用的过程中映像深刻的地方就是搭建一套分布式容器平台很容易上手，使用规则也非常符合新用户的习惯。但是在深度使用Mesos系统之后发现，对于初学者本地环境的多样性问题，Mesos系统并不能快速的解决问题。所以，Apache Mesos的使用体验并没有Docker Swarm那样轻量级别，让开发者快速部署一套Mesos环境还是很困难的事情。这个问题一直到Cisco开源了一套MiniMesos之后才得到一定的缓解。MiniMesos项目是通过Docker Compose编排系统来在本地开发环境自动构建一套Apache Zookeeper + Apache Mesos + Marathon + Consul + Mesos DNS + Registrator全家桶。可以方便开发者快速通过这套Mesos环境一键部署全家桶完成Mesos环境的部署，方便调度框架的二次开发工作。Cisco就是利用这套miniMesos工具快速做出了一个Elasticsearch on mesos应用调度框架。

Mesos社区拥有很多中国开发者，大家也非常热情和耐心帮助Mesos社区的成长，大家通过大量的Mesos使用经实践已经积累了很多案例，所以我期望通过本书的汇总学习，给读者提供一份完整学习体系的Mesos使用手册。在此感谢这些作者的贡献。

* 陈显鹭 - 灵雀云开发工程师\(xianlubird@gmail.com\)
* 徐磊 - 去哪儿系统开发工程师\(49068995@qq.com\)
* 赵英俊 - 城云科技（杭州）有限公司\(zyj@citycloud.com.cn\)
* 周伟涛 - 阿里云\(zhouwtlord@gmail.com）
* 杨成伟 - 爱奇艺\(me@chengweiyang.cn\)

还有最后，作为本书的作者，一直忙于创业、朋友、家庭的事务权衡之中。期间经历了二孩的出生。所以我将把这本书作为礼物献给我的妻子和二个宝宝。谢谢他们对我事业默默的支持。

在写作本书时，安装的所有组件、所用示例和操作等皆基于 **Mesos 1.3.1** 版本。

[文章目录](SUMMARY.md)

GitHub 地址: https://github.com/xiaods/mesos-handbook

Gitbook 在线浏览：https://xiaods.gitbooks.io/mesos-handbook/

## 如何使用本书

**在线浏览**

访问 [gitbook](https://xiaods.gitbooks.io/mesos-handbook/)

**注意**：<u>文中涉及的配置文件和代码链接在 gitbook 中会无法打开，请下载 github
源码后，在 MarkDown
编辑器中打开，点击链接将跳转到你的本地目录，推荐使用[typora](https://www.typora.io)</u>。

**本地查看**

1. 将代码克隆到本地
2. 安装 gitbook：[Setup and Installation of GitBook](https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md)
3. 执行 gitbook serve
4. 在浏览器中访问http://localhost:4000
5. 生成的文档在 `_book` 目录下

## 贡献文档

### 文档的组织规则

- 如果要创建一个大的主题就在最顶层创建一个目录；
- 如果要创建一个大的主题就在最顶层创建一个目录；
- 全书五大主题，每个主题一个目录，其下不再设二级目录；
- 所有的图片都放在最顶层的 `images` 目录下，原则上文章中用到的图片都保存在本地；
- 所有的文档的文件名使用英文命名，可以包含数字和中划线； - `etc`、`manifests`目录专门用来保存配置文件和文档中用到的其他相关文件；

### 添加文档

1. 在该文章相关主题的目录下创建文档；
2. 在 `SUMMARY.md` 中在相应的章节下添加文章链接；
3. 执行 `gitbook serve` 测试是否报错，访问 http://localhost:4000 查看该文档是否出现在相应主题的目录下；
4. 提交PR



## 关于

[贡献者列表](https://github.com/xiaods/mesos-handbook/graphs/contributors)



