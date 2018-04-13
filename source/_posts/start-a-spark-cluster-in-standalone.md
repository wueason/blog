---
title: 搭建一个最简单的Spark集群
date: 2018-04-07 17:12:16
tags:
---

## 搭建一个最简单的Spark集群

### Spark集群的工作原理

![集群工作原理](images/cluster-overview.png "集群工作原理")

`SparkContext`可以连接到几种类型的集群管理器(`Cluster Managers`), 一旦连接，Spark就会获取集群中节点(`Worker Node`)上的执行者(`Executor`)，这些执行者是运行计算并为应用程序存储数据的进程。 然后，它将写好的应用程序代码（JAR包或Python文件）发送给执行者。 最后，SparkContext发送任务给执行者运行。

### 集群类型

1. Standalone
Spark框架本身也自带了完整的资源调度管理服务，可以独立部署到一个集群中，而不需要依赖其他系统来为其提供资源管理调度服务。

2. Apache Mesos
Mesos是一种资源调度管理框架，可以为运行在它上面的Spark提供服务。Spark程序所需要的各种资源，都由Mesos负责调度。目前，Spark官方推荐采用这种模式，所以，许多公司在实际应用中也采用该模式。

3. Hadoop YARN
Spark可运行于YARN之上，与Hadoop进行统一部署，资源管理和调度依赖YARN，分布式存储依赖HDFS。

4. Kubernetes
Kubernetes（k8s）是自动化容器操作的开源平台，这些操作包括部署，调度和节点集群间扩展。

5. 同时也有第三方的集群模式，但尚未得到官方的支持，比如[Nomad](https://github.com/hashicorp/nomad-spark)。

之后本教程将主要使用Standalone模式。

### Standalone模式

+ 启动集群的master服务
	`./sbin/start-master.sh`

+ 启动集群的Workers
	`./sbin/start-slave.sh <master-spark-URL>`，此处的`master-spark-URL`就是刚刚启动好的master服务地址，协议头为`spark:://`，你可以通过`http://localhost:8080`查看。本例中为`spark://ubuntu:7077`或`spark://127.0.0.1:7077`。

+ `jps`，可以查看集群信息（Master/Worker的进程ID）

+ 停止集群的脚本也在`./sbin`下。

