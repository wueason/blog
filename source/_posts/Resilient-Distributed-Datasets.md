---
title: 弹性分布式数据集-RDD
date: 2018-04-09 15:55:30
tags:
---

## 弹性分布式数据集-RDD

弹性分布式数据集（RDD）是一种具有容错特性的数据集合，能在Spark的各个组件间做出各类转换并无缝传递。

有两种方式创建RDD：并行化数据集合或是外部数据集合（文件，HDFS，HBase等）。

### RDD基本操作

#### 并行化集合（Parallelized Collections）

```python
data = [1, 2, 3, 4, 5]
distData = sc.parallelize(data)
```

#### 外部数据集

+ Bash中创建文件
```bash
echo -e "zhangsan, 23\nlisi, 25\nwanger, 27" > /usr/local/spark/data.txt
```

+ Pyspark Shell中创建基于文件的RDD
```python
distFile = sc.textFile("data.txt")
```

当使用文件名时，所以的工作节点（Worker Nodes）都应该有能力访问到该文件。再次强调，本例基于Standalone。

#### RDD基本操作

```python
lines = sc.textFile("data.txt")
lineLengths = lines.map(lambda s: len(s)) # 返回每行的长度作为一个集合
lineLengths.collect() # [12, 8, 10]，返回元素集合
lineLengths.first() # 12，返回第一个元素
lineLengths.take(2) # [12, 8]，，返回第一个元素的集合
lineLengths.count() # 3，集合元素总数
lineLengths.reduce(lambda a, b: a + b) # 30，聚集函数，求和
lineLengthsFiltered = lineLengths.filter(lambda x: x >= 10) # [12, 10]，过滤出长度大于等于10的集合
```

更多可参考[集合转换和操作](http://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations)

#### Spark中的函数传递

Spark API中对函数传递有很大的依赖，主要有三种方式

+ Lambda表达式
```python
def doStuff(self, rdd):
    field = self.field
    return rdd.map(lambda s: field + s)
```

+ 内部定义的函数
```python
if __name__ == "__main__":
    def myFunc(s):
        words = s.split(" ")
        return len(words)

    sc = SparkContext(...)
    sc.textFile("file.txt").map(myFunc)
```

+ 模块中定义的函数
