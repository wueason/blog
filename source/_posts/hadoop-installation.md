---
title: Hadoop安装
date: 2018-04-21 13:11:43
tags: [Hadoop]
---

### 环境依赖
+ Java，具体可参考[推荐版本](http://wiki.apache.org/hadoop/HadoopJavaVersions)
+ ssh，sshd服务

### Java 1.8
+ `sudo apt-get update`
+ `sudo apt-get install -y default-jre default-jdk`


### Hadoop 2.7.5
+ `wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.5/hadoop-2.7.5.tar.gz`
+ `tar zxvf hadoop-2.7.5.tar.gz`
+ `sudo mv hadoop-2.7.5 /usr/local/hadoop`
+ `sudo chown vagrant /usr/local/hadoop`


### PATH
+ `vi ~/.bashrc`，添加以下代码
    ```
export JAVA_HOME=/usr/lib/jvm/default-java
export HADOOP_HOME=/usr/local/hadoop
    ```

### Hadoop目前支持的集群模式
+ 单机模式安装
+ 伪分布式模式安装
+ 完全分布式安装

### 单机模式安装
+ `mkdir input`
+ `cp etc/hadoop/*.xml input`
+ `bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.6.jar grep input output 'dfs[a-z.]+'`
+ `ls output`

### 伪分布式模式安装

+ shh配置

	- `ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa`
	- `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
	- `chmod 0600 ~/.ssh/authorized_keys`
	- 测试：`ssh localhost`

+ 配置文件

	- etc/hadoop/core-site.xml，ubuntu-xenial为服务器主机名或IP：

	```
	<configuration>
	    <property>
	        <name>fs.defaultFS</name>
	        <value>hdfs://ubuntu-xenial:9000</value>
	    </property>
	</configuration>
	```

	- etc/hadoop/hdfs-site.xml：

	```
	<configuration>
	    <property>
	        <name>dfs.replication</name>
	        <value>1</value>
	    </property>
	</configuration>
	```

	- etc/hadoop/hadoop-env.sh文件中，修改：
	`export JAVA_HOME=/usr/lib/jvm/default-java`

+ 运行：

	1. 格式化文件系统
	`bin/hdfs namenode -format`

	2. 运行NameNode和DataNode服务
	`sbin/start-dfs.sh`

	运行日志默认写在`$HADOOP_LOG_DIR`文件夹下，默认为`$HADOOP_HOME/logs`

	3. NameNode web界面
	`http://ubuntu-xenial:50070/`

	4. 建立分布式文件夹
	`bin/hdfs dfs -mkdir /user`
	`bin/hdfs dfs -mkdir /user/<username>`

	5. 将本地文件复制到分布式系统
	`bin/hdfs dfs -put etc/hadoop input`

	6. 运行MapReduce任务
	`bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.6.jar grep input output 'dfs[a-z.]+'`

	7. 从分布式文件系统文件复制到本地，并验证
	`bin/hdfs dfs -get output output`
	`cat output/*`
	or
	`bin/hdfs dfs -cat output/*`

	8. 停止服务
	`sbin/stop-dfs.sh`

### YARN服务的引入

+ 配置文件
	- etc/hadoop/mapred-site.xml：
	```
	<configuration>
	    <property>
	        <name>mapreduce.framework.name</name>
	        <value>yarn</value>
	    </property>
	</configuration>
	```

	- etc/hadoop/yarn-site.xml：
	```
	<configuration>
	    <property>
	        <name>yarn.nodemanager.aux-services</name>
	        <value>mapreduce_shuffle</value>
	    </property>
	</configuration>
	```

+ 运行YARN的ResourceManager和NodeManager
`sbin/start-yarn.sh`

+ ResourceManager的web界面
`http://ubuntu-xenial:8088/`

+ 运行MapReduce任务

+ 停止YARN服务
`sbin/stop-yarn.sh`
