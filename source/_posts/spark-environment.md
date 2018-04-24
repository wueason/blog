---
title: Spark环境搭建
date: 2018-04-04 09:22:42
tags: [Spark,环境搭建]
---

### 目标：
+ 搭建Spark环境
+ 写出第一个Spark程序并运行。

### 运行环境
+ ubuntu 16.04
+ python3
+ Java 1.8
+ Spark 2.1.0
+ Hadoop 2.7.5

### ubuntu 16.04
+ 基于[vagrant](https://www.vagrantup.com/)
+ `vagrant box add ubuntu/xenial https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-vagrant.box`
+ `vagrant init ubuntu/xenial`
+ 增加ip地址解析`config.vm.network "public_network", ip: "192.168.100.110"`
+ 增加内存上限`config.vm.provider "virtualbox" do |vb|    	vb.memory = "5248"		end`
+ `vagrant up`
+ `vagrant ssh`

### python3
+ ~~`curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"`~~
+ ~~`python3 get-pip.py`~~
+ ~~`sudo pip install ipython`~~
+ `sudo apt install python3-pip`
+ `sudo pip3 install ipython`
+ `sudo pip3 install numpy`

### Java 1.8
+ `sudo apt-get update`
+ `sudo apt-get install -y default-jre default-jdk`

### Hadoop 2.7.5
+ `wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.5/hadoop-2.7.5.tar.gz`
+ `tar zxvf hadoop-2.7.5.tar.gz`
+ `sudo mv hadoop-2.7.5 /usr/local/hadoop`
+ `sudo chown vagrant /usr/local/hadoop`

### Spark 2.3.0
+ `wget https://www.apache.org/dyn/closer.lua/spark/spark-2.3.0/spark-2.3.0-bin-without-hadoop.tgz`
+ `tar zxvf spark-2.3.0-bin-without-hadoop.tgz`
+ `sudo mv spark-2.3.0-bin-without-hadoop /usr/local/spark`
+ `sudo chown vagrant /usr/local/spark`
+ 在文件`/usr/local/spark/conf/spark-env.sh`文件开头增加:`export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)`

### PATH
+ `vi ~/.bashrc`，添加以下代码
    ```
export PYSPARK_DRIVER_PYTHON=ipython
export JAVA_HOME=/usr/lib/jvm/default-java
export HADOOP_HOME=/usr/local/hadoop
export SPARK_HOME=/usr/local/spark
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.6-src.zip:$PYTHONPATH
export PYSPARK_PYTHON=python3
export PATH=$HADOOP_HOME/bin:$SPARK_HOME/bin:/usr/local/hadoop/sbin:/usr/local/hadoop/bin:$PATH
    ```

### Spark 初体验

+ `cd /usr/local/spark`
+ 执行`./bin/pyspark`，进入Spark shell。没有报错，表明安装正确。
+ 运行Spark自带的例子。`./bin/spark-submit examples/src/main/python/pi.py 10`


### 第一个Spark程序

+ `vi /usr/local/spark/code/wordCount.py`

    ```Python
from pyspark import SparkContext
sc = SparkContext( 'local', 'test')
textFile = sc.textFile("file:///usr/local/spark/README.md")
wordCount = textFile.flatMap(lambda line: line.split(" ")).map(lambda word: (word,1)).reduceByKey(lambda a, b : a + b)
wordCount.foreach(print)
    ```

**至此，本文也到了该完结的时候，在敲下这个命令之后：`python3 code/wordCount.py`**