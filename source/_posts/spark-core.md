---
title: Spark Core
date: 2020-04-20 22:51:02
tags: [Spark Core]
---

### 基本操作

```
PYSPARK_DRIVER_PYTHON=ipython ./bin/pyspark --master local[4]


from pyspark import SparkContext, SparkConf

conf = SparkConf().setAppName('myspark').setMaster("local[4]")
sc = SparkContext(conf=conf)


PySpark 支持 Hadoop, local file system, HDFS, Cassandra, HBase, Amazon S3, etc. Spark supports text files, SequenceFiles, and any other Hadoop InputFormat.


data = [1, 2, 3, 4, 5]
# 多cpu并行计算，如sc.parallelize(data, 4)
distData = sc.parallelize(data)
distData.reduce(lambda a, b: a + b)


distFile = sc.textFile("README.md")
# 计算行数
distFile.map(lambda s: len(s)).reduce(lambda a, b: a + b)


rdd = sc.parallelize(range(1, 4)).map(lambda x: (x, "a" * x))
rdd.saveAsSequenceFile("1.txt")
sorted(sc.sequenceFile("1.txt").collect())


./bin/pyspark --jars /path/to/elasticsearch-hadoop.jar

conf = {"es.resource" : "index/type"}  # assume Elasticsearch is running on localhost defaults
rdd = sc.newAPIHadoopRDD("org.elasticsearch.hadoop.mr.EsInputFormat",
                             "org.apache.hadoop.io.NullWritable",
                             "org.elasticsearch.hadoop.mr.LinkedMapWritable",
                             conf=conf)
rdd.first()  # the result is a MapWritable that is converted to a Python dict
(u'Elasticsearch ID',
 {u'field1': True,
  u'field2': u'Some Text',
  u'field3': 12345})


lines = sc.textFile("data.txt")
lineLengths = lines.map(lambda s: len(s))
# 等下还需要使用时，可以持久化
lineLengths.persist()
totalLength = lineLengths.reduce(lambda a, b: a + b)


# 不能使用全局变量 global，应该使用accumulator
accum = sc.accumulator(0)
sc.parallelize([1, 2, 3, 4]).foreach(lambda x: accum.add(x))
accum.value  #100


rdd.collect().foreach(println)  #这样打印有可能内存溢出
#打印少数元素
rdd.take(100).foreach(println)


pairs = sc.parallelize([1, 2, 3, 4]).map(lambda s: (s, 1))
counts = pairs.reduceByKey(lambda a, b: a + b)

```