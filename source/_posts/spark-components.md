---
title: Spark组件
date: 2018-04-06 08:55:02
tags:
---

## Spark组件

+ Spark Core：Spark Core包含Spark的基本功能，如内存计算、任务调度、部署模式、故障恢复、存储管理等。Spark建立在统一的抽象RDD之上，使其可以以基本一致的方式应对不同的大数据处理场景。

+ Spark SQL：Spark SQL允许开发人员直接处理RDD，同时也可查询Hive、HBase等外部数据源。Spark SQL的一个重要特点是其能够统一处理关系表和RDD，使得开发人员可以轻松地使用SQL命令进行查询，并进行更复杂的数据分析。

+ Spark Streaming：Spark Streaming支持高吞吐量、可容错处理的实时流数据处理。

+ MLlib（机器学习）：MLlib提供了常用机器学习算法的实现，包括聚类、分类、回归、协同过滤等，降低了机器学习的门槛，开发人员只要具备一定的理论知识就能进行机器学习的工作；

+ GraphX（图计算）：GraphX是Spark中用于图计算的API。

本教程将专注于MLlib组件的学习。