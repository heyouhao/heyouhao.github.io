---
layout: post
title: Hadoop2.x伪分布式环境搭建
date: 2019-08-02 08:25:20
comments: true
tags: 
  - hadoop
---

### 1） hadoop 环境准备测试:
#### a. ip地址
![1.jpg-98.6kB][1]
<!--more-->
#### b. ping通外网
![1.jpg-66.9kB][2]
#### c. 主机名
![1.jpg-10.7kB][3]
#### d. 关闭防火墙
![1.jpg-32.7kB][4]
#### f. 创建hadoop目录
![1.jpg-56.3kB][5]
#### g. jdk环境配置
##### 查看系统jdk
![1.jpg-24.8kB][6]
##### 卸载jdk
![2.jpg-23.9kB][7]
##### 安装jdk
![4.jpg-11.1kB][8]
##### 配置环境变量
![1.jpg-28.6kB][9]
##### 刷新测试
![2.jpg-32.5kB][10]
### 2） HDFS 配置、 启动命令、 测试命令
#### a. HDFS配置
##### 解压hadoop压缩文件
![1.jpg-11.1kB][11]
```linux
[hadoop@pseudo opt]$ tar -zxvf /opt/softwares/hadoop-2.5.0.tar.gz -C /opt/modules/hadoop/
```
##### 修改hadoop-env.sh，yarn-env.sh，mapred-env.sh配置文件，加入以下代码
```linux
export JAVA_HOME=/opt/modules/jdk1.7.0_67
```
hadoop-env.sh 额外修改
```linux
export HADOOP_PREFIX=/opt/modules/hadoop/hadoop-2.5.0
export HADOOP_CONF_DIR=/opt/modules/hadoop/hadoop-2.5.0/etc/hadoop
```
##### 修改core-site.xml配置文件,如下显示
![1.jpg-38.6kB][12]
```xml
<configuration>
   <property>
	    <name>fs.defaultFS</name>
	    <value>hdfs://pseudo:8020</value>
   </property>
   <property>
		<name>hadoop.tmp.dir</name>
		<value>/opt/modules/hadoop-2.5.0/data/tmp</value>
	</property>
</configuration>
```
##### 修改hdfs-site.xml配置文件,如下显示
![1.jpg-17.8kB][13]
```xml
<configuration>
   <property>
		<name>dfs.replication</name>
		<value>1</value>
	</property>
</configuration>
```
##### 首次启动hadoop格式化namenode
![1.jpg-7.4kB][14]
```linux
[hadoop@pseudo hadoop-2.5.0]$ bin/hdfs namenode -format
```
![2.jpg-99kB][15]
#### b. 启动命令
##### 启动namenode
![1.jpg-22kB][16]
```linux
[hadoop@pseudo hadoop-2.5.0]$ sbin/hadoop-daemon.sh start namenode
```
##### 启动datanode
![2.jpg-24.6kB][17]
```linux
[hadoop@pseudo hadoop-2.5.0]$ sbin/hadoop-daemon.sh start datanode
```
#### c. 测试命令
![1.jpg-14.2kB][18]

![1.jpg-66.9kB][19]
### 3） YARN 配置、 启动命令、 WEB UI 页面
#### a. YARN 配置
##### 修改yarn-site.xml配置文件,如下显示
![1.jpg-48kB][20]
```xml
<configuration>
<!-- Site specific YARN configuration properties -->
    <property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>pseudo</value>
	</property>
</configuration>
```
#### b. 启动命令
##### 启动resourcemanager
![1.jpg-27.3kB][21]
```linux
[hadoop@pseudo hadoop-2.5.0]$ sbin/yarn-daemon.sh start resourcemanager
```
##### 启动nodemanager
![2.jpg-27.8kB][22]
```linux
[hadoop@pseudo hadoop-2.5.0]$ sbin/yarn-daemon.s start nodemanager
```
#### c. WEB UI 页面
![1.jpg-62.5kB][23]
### 4） MapReduce 配置、 案例 WordCount 测试运行、 如何提交 Job、查看运行结果
#### a. MapReduce 配置
##### 将mapred-site.xml.template复制为mapred-site.xml配置文件,如下显示
```linux
[hadoop@pseudo hadoop]$ cp mapred-site.xml.template mapred-site.xml
```
![1.jpg-20.9kB][24]
```xml
<configuration>
     <property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```
#### b. 案例 WordCount 测试运行
##### 创建input文件并在hdfs上创建/user/hadoop目录
```linux
[hadoop@pseudo ~]$ touch input
[hadoop@pseudo ~]$ cat input 
hello hadoop
hadoop hello
hello heyouhao 
```
![1.jpg-67.3kB][25]
##### 将文件上传到hdfs上并查看input文件
![1.jpg-121kB][26]
```linux
[hadoop@pseudo hadoop-2.5.0]$ bin/hdfs dfs -put /home/hadoop/input /user/hadoop
[hadoop@pseudo hadoop-2.5.0]$ bin/hdfs dfs -ls /user/hadoop
```
#### c. 提交 Job
![1.jpg-22.6kB][27]
```linux
[hadoop@pseudo hadoop-2.5.0]$ bin/yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar wordcount /user/hadoop/input /user/hadoop/output
```
#### d. 查看运行结果
##### 代码查看
![1.jpg-78.2kB][28]
##### hdfs上文件查看
![1.jpg-128.6kB][29]
```linux
[hadoop@pseudo hadoop-2.5.0]$ bin/hdfs dfs -ls /user/hadoop/output
[hadoop@pseudo hadoop-2.5.0]$  bin/hdfs dfs -cat /user/hadoop/output/part-r-00000
```
### 5） 根据自己理解的 HDFS、 YARN 及 MapReduce 功能进行描述
HDFS：hdfs是分布式文件系统，主要用于hadoop的文件存储，分布在不同的廉价机器上,主(namenode)从(datanode)架构，文件以block块的形式存储在datanode上，由namenode记录元数据信息，每台机器ip对应的datanode以及每台datanode是否可用等信息。

YARN：yarn是一个资源管理以及任务调度者，同样主(ResourceManager)从(NodeManager)架构，由客户端提交任务给ResourceManager，由ResourceManager分配到相应的NodeManager节点上的ApplicationMaster应用程序处理。

MapReduce：mapReduce是一个离线批处理计算框架，分为3个阶段处理，map、shuffle、reduce，由多个map同步处理任务，在通过shuffle交给reduce整合处理。


  [1]: http://static.zybuluo.com/myouhao/kkfumo7v65r5mm1qzs12a4df/1.jpg
  [2]: http://static.zybuluo.com/myouhao/lgduz292bw4e00mhpdip7d6j/1.jpg
  [3]: http://static.zybuluo.com/myouhao/q3shvas97xh39zz22u5oqjxm/1.jpg
  [4]: http://static.zybuluo.com/myouhao/hvebf71f2l0ik9ykkryr40b4/1.jpg
  [5]: http://static.zybuluo.com/myouhao/u1zxtsz1a0vrh39tbnok0dib/1.jpg
  [6]: http://static.zybuluo.com/myouhao/jzll884worji8fgmcfcc6om3/1.jpg
  [7]: http://static.zybuluo.com/myouhao/bvow23vs4evul5ty3xidl7cf/2.jpg
  [8]: http://static.zybuluo.com/myouhao/mcc55n0ty3odiiwfpdfvwdsa/4.jpg
  [9]: http://static.zybuluo.com/myouhao/ksuttqcswpczgmxnm9tmua7j/1.jpg
  [10]: http://static.zybuluo.com/myouhao/vnmjt8nsa1y0wgwnlgg55864/2.jpg
  [11]: http://static.zybuluo.com/myouhao/y9phcrbyoo6mq0hjn3gj3vt2/1.jpg
  [12]: http://static.zybuluo.com/myouhao/bckpgh4e1v8o7ywvl20p1hzi/1.jpg
  [13]: http://static.zybuluo.com/myouhao/c9sokqczjrdzdev9f8o7ftpn/1.jpg
  [14]: http://static.zybuluo.com/myouhao/loueacz86wdjomesjeh07yx5/1.jpg
  [15]: http://static.zybuluo.com/myouhao/cjw6dgpgwgimzt4enyhgrg85/2.jpg
  [16]: http://static.zybuluo.com/myouhao/5ic4an19mtsufsxtmhg6a5ex/1.jpg
  [17]: http://static.zybuluo.com/myouhao/hqw2d8bpf98ydj0eh21643x5/2.jpg
  [18]: http://static.zybuluo.com/myouhao/9f3k2fjuzlgacr5dwv9nudta/1.jpg
  [19]: http://static.zybuluo.com/myouhao/0ec9jttg1l3iz0euo9kwznn1/1.jpg
  [20]: http://static.zybuluo.com/myouhao/mgygpwu7njsslbqepxfay78z/1.jpg
  [21]: http://static.zybuluo.com/myouhao/o0qcr8pyfguxyvvsqpbahay2/1.jpg
  [22]: http://static.zybuluo.com/myouhao/8yfehaowbfvdezzquaotbgau/2.jpg
  [23]: http://static.zybuluo.com/myouhao/rvsp0h14f13imx66bvb8ckvj/1.jpg
  [24]: http://static.zybuluo.com/myouhao/fezvs2pq2jhi97j3ly3zdx47/1.jpg
  [25]: http://static.zybuluo.com/myouhao/ptw02hdt9z04cx01qm36bkv6/1.jpg
  [26]: http://static.zybuluo.com/myouhao/hqngs6wr92ct6dn18r0oua0g/1.jpg
  [27]: http://static.zybuluo.com/myouhao/urmmox07pdma758lnvl3mvw7/1.jpg
  [28]: http://static.zybuluo.com/myouhao/vkurklajj4r4u6qrni1iz397/1.jpg
  [29]: http://static.zybuluo.com/myouhao/oy74go8zqqb9gocn9wi712d1/1.jpg