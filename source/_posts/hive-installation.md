---
title: 分布式数据仓库Hive
date: 2018-04-25 15:11:43
tags: [Hadoop,Hive,数据仓库]
---

### 简介

Hive是基于Hadoop的数据仓库解决方案。由于Hadoop本身在数据存储和计算方面有很好的可扩展性和高容错性，因此使用Hive构建的数据仓库也秉承了这些特性。

简单来说，Hive就是在Hadoop上架了一层SQL接口，可以将SQL翻译成MapReduce去Hadoop上执行，这样就使得数据开发和分析人员很方便的使用SQL来完成海量数据的统计和分析，而不必使用编程语言开发MapReduce那么麻烦。

### 环境依赖
+ Java
+ Hadoop，可移步至[Hadoop安装](https://oobspark.github.io/2018/04/21/hadoop-installation/)

### Java 1.8
+ `sudo apt-get update`
+ `sudo apt-get install -y default-jre default-jdk`

### Hive 2.3.3
+ `wget http://mirrors.hust.edu.cn/apache/hive/hive-2.3.3/apache-hive-2.3.3-bin.tar.gz`
+ `tar zxvf apache-hive-2.3.3-bin.tar.gz`
+ `sudo mv apache-hive-2.3.3-bin /usr/local/hadoop/hive`
+ `sudo chown vagrant:vagrant /usr/local/hadoop/hive`

### PATH
`vi ~/.bashrc`，添加以下代码
```
export JAVA_HOME=/usr/lib/jvm/default-java
export HIVE_HOME=/usr/local/hadoop/hive
export PATH=$HIVE_HOME/bin:$PATH
export HADOOP_HOME=<hadoop-install-dir>
```

### HDFS上的文件初始化
  `$HADOOP_HOME/bin/hdfs dfs -mkdir       /tmp`
  `$HADOOP_HOME/bin/hdfs dfs -mkdir       /user/hive/warehouse`
  `$HADOOP_HOME/bin/hdfs dfs -chmod g+w   /tmp`
  `$HADOOP_HOME/bin/hdfs dfs -chmod g+w   /user/hive/warehouse`

### Hive元数据的储存模式
+“单用户”模式：利用该模式连接到内存数据库Derby，Derby不支持多会话连接，这种模式一般用于单机测试
+“多用户”模式：通过网络和JDBC连接到规模更大的专业数据库，如MySQL
+“远程服务器”模式：用于非Java客户端访问在远程服务器上储存的元数据库，需要在服务端启动一个MetaStoreServer，然后在客户端通过Thrift协议访问该服务器，进而访问到元数据

### 单用户模式

+ 配置文件
`cp conf/hive-default.xml.template  conf/hive-default.xml`

+ Hive Shell：
`bin/hive`

### Hive集成MySQL数据库

1. 新建hive数据库，用来保存hive的元数据
`mysql> create database hive;`

2. 将hive数据库下的所有表的所有权限赋给hadoop用户，并配置mysql为hive-site.xml中的连接密码，然后刷新系统权限关系表
```
mysql> CREATE USER  'hadoop'@'%'  IDENTIFIED BY 'mysql';
mysql> GRANT ALL PRIVILEGES ON  *.* TO 'hadoop'@'%' WITH GRANT OPTION;
mysql> flush privileges;
```

### 多用户模式安装


+ `vi conf/hbase-site.xml`，没有的话新建
```
<configuration>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true</value>
		<description>metadata is stored in a MySQL server</description>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
		<description>MySQL JDBC driver class</description>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>hive</value>
		<description>user name for connecting to mysql server</description>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>hive</value>
		<description>password for connecting to mysql server</description>
	</property>
</configuration>
```

+ [下载](https://dev.mysql.com/downloads/connector/j/)相应的mysql jar包，解压后放入$HIVE_HIME/lib下
`lib/mysql-connector-java-5.1.46-bin.jar`

+ 初始化数据表
`schematool -dbType mysql -initSchema`
`schematool -dbType mysql -info`

+ Hive Shell：
	`bin/hive`

	```hive
	CREATE TABLE pokes (foo INT, bar STRING);
	CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);
	SHOW TABLES;
	SHOW TABLES '.*s';
	DESCRIBE invites;
	LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
	LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
	LOAD DATA LOCAL INPATH './examples/files/kv3.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-08');
	SELECT a.foo FROM invites a WHERE a.ds='2008-08-15';
	INSERT OVERWRITE DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM invites a WHERE a.ds='2008-08-15';
	```

	[DDL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL) 和 [DML](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML)