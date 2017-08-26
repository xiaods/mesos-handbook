# 运行 Spark 任务

在上一节中我们介绍了怎样搭建一个运行在 Mesos 集群之上的 Spark 服务，并且可以通过 ZooKeeper 实现 Spark 服务的高可用性。

本节将介绍怎样提交 Spark 任务，以及怎样将 Spark 任务运行在 Docker 中。

Spark 提供了多种编程语言绑定，包括：Java, Scala, Python, R 等。介于目前 Java 在大数据领域的绝对优势，这里介绍怎样提交 Java 任务，另外，由于 Scala 也是一种 JVM 语言，提交 Scala 任务和提交 Java 任务的方式类似。

## spark-submit

Spark 提供了一个工具用来提交任务到 Spark 服务中，即：spark-submit，在 `bin` 目录下。spark-submit 支持许多参数来定制提交任务的行为，包括一些必要的参数和可选的参数。

`spark-submit --help` 能够打印出 spark-submit 的帮助文档，下面介绍一些常用的参数。

参数 | 含义
------ | -----
`--master MASTER_URL` | 指定 spark 服务地址，例如：mesos://10.23.85.234:7077
`--deploy-mode DEPLOY_MODE` | 指定提交任务的方式，合法值为 client 或者 cluster，默认为 client
`--class CLASS_NAME` | 指定将要被运行的 Java/Scala class 名称
`--name NAME` | 任务的名称
`--conf PROP=VALUE` | 设置 Spark 属性
`--properties-file FILE` | 指定 Spark 配置文件，默认为 conf/spark-defaults.conf
`--driver-memory MEM` | 指定 spark driver 使用的内存，默认为 1024M
`--executor-memory MEM` | 指定 spark executor 使用的内存，默认为 1G

以下参数在 standalone 和 Mesos 集群方式下有效。

参数 | 含义
------ | -----
`--total-executor-cores NUM` | 所有 executor 占用的 cpu 核数

以下参数只有在 standalone 和 Mesos 集群方式，并且在 `--deploy-mode` 为 cluster 时才有效。

参数 | 含义
------ | -----
`--supervise` | 如果指定，当 spark driver 退出时会被自动重启
`--kill SUBMISSION_ID` | 停止任务
`--status SUBMISSION_ID` | 查看任务状态

## 提交任务

在学习了 spark-submit 常用参数后，下面使用 spark-submit 来提交任务。提交一个 Java/Scala 任务，需要指定以下参数：

  - `--master`
  - `--class`

以提交 SparkPi 这个示例应用为例，在 10.23.85.235 上 /home/spark/spark-1.5.2-bin-hadoop2.6/bin 目录中，执行下面的命令。

```
$ ./spark-submit --master mesos://10.23.85.234:7077 --class org.apache.spark.examples.SparkPi --name "Spark PI example" ../lib/spark-examples-1.5.2-hadoop2.6.0.jar 100
FIXME: output
```

上面的命令首先指定了 spark 服务地址，这里是上节我们搭建的 spark on mesos 服务地址，然后指定了要运行的类名称，然后还指定了本次任务的名称，最后指定了包含任务内容的 jar 文件。

spark-submit 默认会以 client 的方式提交任务，所以 spark driver 会在本地前台执行，任务的输出会直接通过终端打印出来。

如果修改上面的命令，添加 `--deploy-mode cluster` 参数，那么任务将会以 cluster 的方式运行，如下所示：

```
$ ./spark-submit --master mesos://10.23.85.234:7077 --class org.apache.spark.examples.SparkPi --name "Spark PI example" --deploy-mode cluster ../lib/spark-examples-1.5.2-hadoop2.6.0.jar 100
FIXME: output
```

通常来说，对于运行在 Mesos 上的 Spark 任务，我们都可以指定 executor 能够使用的资源上限，避免 spark 任务占用过多的资源，例如：

```
$ ./spark-submit --master mesos://10.23.85.234:7077 --class org.apache.spark.examples.SparkPi --name "Spark PI example" --deploy-mode cluster --total-executor-cores 10 ../lib/spark-examples-1.5.2-hadoop2.6.0.jar 100
FIXME: output
```

由于 executor 默认会占用 1GB 内存，1 核 CPU，所以本次提交最多占用 10 核 CPU，10GB 内存。

对于 Spark 任务来说，通常需要指定较大的 executor 内存，因为 Spark 专门为内存计算设计，在较大内存中更容易获得更快的执行速度，例如：为每个 executor 指定 20GB 内存大小，甚至更大，由于本书的实验环境虚拟机内存大小为 16GB，所以不宜指定太大的 executor 内存。

还需要注意的是，spark 任务的 jar 应该要包含除 spark 之外的其它所有依赖，以免集群中的节点并没有配置所有的依赖包。

## 集成 Docker

