---
title: Hbase安装
date: 2018-04-23 15:11:43
tags: [Hadoop,Hbase]
---

### 简介

HBase – Hadoop Database，是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang 所撰写的Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。HBase是Apache的Hadoop项目的子项目。

HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列的而不是基于行的模式。

HBase是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。

### 环境依赖
+ Java，具体可参考[推荐版本](http://hbase.apache.org/book.html#java)
+ ssh，sshd服务

### Java 1.8
+ `sudo apt-get update`
+ `sudo apt-get install -y default-jre default-jdk`

### Hbase 1.3.2
+ `wget http://mirrors.hust.edu.cn/apache/hbase/1.3.2/hbase-1.3.2-bin.tar.gz`
+ `tar zxvf hbase-1.3.2-bin.tar.gz`
+ `sudo mv hbase-1.3.2 /usr/local/hadoop/hbase`
+ `sudo chown vagrant:vagrant /usr/local/hadoop`，注意此处，必须为hbase上级目录设置权限

### PATH
+ `vi ~/.bashrc`，添加以下代码
    ```
export JAVA_HOME=/usr/lib/jvm/default-java
export HBASE_HOME=/usr/local/hadoop/hbase
    ```

### 单机模式安装

+ 配置文件

	- hbase-site.xml：

	```
	<configuration>
	  <property>
	    <name>hbase.rootdir</name>
	    <value>file:///opt/data/hbase</value>
	  </property>
	  <property>
	    <name>hbase.zookeeper.property.dataDir</name>
	    <value>/opt/data/zookeeper</value>
	  </property>
	</configuration>
	```

	- conf/hbase-env.sh文件中，修改：
	`export JAVA_HOME=/usr/lib/jvm/default-java`
	`export HBASE_MANAGES_ZK=true`，使用hbase自带zookeeper

+ 运行：

	1. 启动服务
	`bin/start-hbase.sh`

	2. 停止服务
	`bin/stop-hbase.sh`

### 伪分布式安装

+ 配置文件

	- hbase-site.xml：

	```
	<configuration>
	  	<property>
	    	<name>hbase.rootdir</name>
	    	<value>hdfs://localhost:8020/hbase</value>
	  	</property>
	  	<property>
	    	<name>hbase.zookeeper.property.dataDir</name>
	    	<value>/opt/data/zookeeper</value>
	  	</property>
	  	<property>
		  <name>hbase.cluster.distributed</name>
		  <value>true</value>
		</property>
	</configuration>
	```

	- conf/hbase-env.sh文件中，修改：
	`export JAVA_HOME=/usr/lib/jvm/default-java`
	`export  HBASE_MANAGES_ZK=true`，使用hbase自带zookeeper

+ 运行：

	1. 启动服务
	`bin/start-hbase.sh`

	2. master副本
	`bin/local-master-backup.sh start 2 3 5`

	3. region server副本
	`bin/local-regionservers.sh start 2 3`

	4. 停止服务
	`bin/stop-hbase.sh`
	`cat /tmp/hbase-vagrant-1-master.pid |xargs kill -9`
	`bin/local-regionservers.sh stop 3`

### HBase基本操作

1. 连接到HBase
```
$ ./bin/hbase shell
hbase(main):001:0>
```

2. 显示帮助，`help`

3. 建表
```
hbase(main):001:0> create 'test', 'cf'
0 row(s) in 0.4170 seconds
```

4. 查看表信息
```
hbase(main):002:0> list 'test'
TABLE
test
1 row(s) in 0.0180 seconds
```

5. 写数据
```
hbase(main):003:0> put 'test', 'row1', 'cf:a', 'value1'
0 row(s) in 0.0850 seconds

hbase(main):004:0> put 'test', 'row2', 'cf:b', 'value2'
0 row(s) in 0.0110 seconds

hbase(main):005:0> put 'test', 'row3', 'cf:c', 'value3'
0 row(s) in 0.0100 seconds
```

6. 获取数据
```
hbase(main):006:0> scan 'test'
ROW                                      COLUMN+CELL
 row1                                    column=cf:a, timestamp=1421762485768, value=value1
 row2                                    column=cf:b, timestamp=1421762491785, value=value2
 row3                                    column=cf:c, timestamp=1421762496210, value=value3
3 row(s) in 0.0230 seconds
```

7. 获取单行数据
```
hbase(main):007:0> get 'test', 'row1'
COLUMN                                   CELL
 cf:a                                    timestamp=1421762485768, value=value1
1 row(s) in 0.0350 seconds
```

8. 禁用表
```
hbase(main):008:0> disable 'test'
0 row(s) in 1.1820 seconds

hbase(main):009:0> enable 'test'
0 row(s) in 0.1770 seconds
```


9. 删除表
```
hbase(main):010:0> drop 'test'
0 row(s) in 0.1370 seconds
```


8. 退出
```
hbase(main):011:0> exit
```