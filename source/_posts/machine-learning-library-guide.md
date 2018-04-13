---
title: Spark MLlib概述
date: 2018-04-08 10:52:28
tags:
---

## Spark MLlib概述

Spark的MLlib，其目标是让机器学习实践更加简单且具可扩展性。提供的特性如下：

+ 机器学习算法：分类，回归，聚类和协同过滤
+ 特征提取：特征提取、转化，将维和选择
+ 工作流（Pipelines）：构建，评估和调整机器学习工作流的工具
+ 数据持久化：持久化/加载模型，算法和工作流
+ 工具集：线性代数，统计学，数据操作的工具支持

### MLlib的主要API

值得注意的是，进入了Spark2.0版本之后，spark.mllib中基于RDD的API（ RDD-based APIs）也进入了维护模式，取而代之的是spark.ml中基于DataFrame的API（DataFrame-based API）。

![RDD-DataFrame](images/RDD-DataFrame.jpg "RDD-DataFrame")

### 依赖

+ 线性代数库`Breeze`
+ 基础线性代数子程序库[`Intel MKL`](https://software.intel.com/en-us/mkl)或[`OpenBLAS`](http://www.openblas.net/)
+ `NumPy >=1.4`

