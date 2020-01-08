---
layout: post
title: 编译 Hadoop 2.x 源码
date: 2019-08-02 08:30:20
comments: true
tags: 
  - hadoop
---

自己动手编译 Hadoop 2.x 源码。
##1、以下几点注意事项：
1、 整个过程以截图、 文字和命令描述进行
2、 编译 Hadoop 2.x 很多注意事项，尤其是准备环境，网络是关键
3、 编译过程中，如果遇到错误，再重新进行执行编译命令（多数错误是由于网络下载依赖 JAR 包导致）

##2、Hadoop 2.x 源码编译步骤
<!--more-->
###1） 环境准备
####a、默认jdk环境配置完成
 ![1.jpg-27.4kB][1]
####b、默认hadoop 2.x 伪分布式环境已经配置完成
 ![1.jpg-17.2kB][2]
####c、ping同网络
 ![1.jpg-48.9kB][3]
####d、查看hdfs命令是否有警告,有则源码不可用
 ![1.jpg-53.6kB][4]
###2）配置maven,并使profile文件生效
####a、解压maven压缩包
![1.jpg-42.8kB][5]
```linux
[hadoop@pseudo softwares]$ tar -zxf apache-maven-3.0.5-bin.tar.gz -C /opt/modules/
```
####b、maven环境变量设置
![1.jpg-40.6kB][6]
```linux
#MAVEN_HOME
export MAVEN_HOME=/opt/modules/apache-maven-3.0.5
export PATH=$PATH:$MAVEN_HOME/bin
```
####c、maven版本测试
![1.jpg-53.6kB][7]
###3) 安装gcc/gcc-c++/make(注意在root上操作)
![1.jpg-43.5kB][8]
```linux
[root@pseudo opt]# yum -y install gcc gcc-c++
```
###4) 安装protobuf
####a、解压编译
![1.jpg-31.4kB][9]

![1.jpg-17kB][10]

![1.jpg-30.9kB][11]
```linux
[root@pseudo softwares]# tar -zxf protobuf-2.5.0.tar.gz -C /opt/modules/
[root@pseudo softwares]# cd /opt/modules/protobuf-2.5.0/
[root@pseudo protobuf-2.5.0]# ./configure --prefix=/usr/local/protoc  /* 这个目录可以随便自己建立，编译后的文件位于/usr/local/protoc/目录下*/
[root@pseudo protobuf-2.5.0]# make
```
####b、查看编译结果
![1.jpg-10.6kB][12]
`注意：如果protoc目录没有需创建`
####c、protobuf环境变量配置
![1.jpg-39.6kB][13]
```linux
[hadoop@pseudo protobuf-2.5.0]$ sudo vi /etc/profile
[sudo] password for hadoop: 
[hadoop@pseudo protobuf-2.5.0]$ tail -3 /etc/profile
#PROTOBUF_HOME
export PROTOBUF_HOME=/usr/local/protoc
export PATH=$PATH:$PROTOBUF_HOME/bin
[hadoop@pseudo protobuf-2.5.0]$ source /etc/profile
```
####d、查看protobuf安装结果
![1.jpg-9.8kB][14]
###5) 下载安装CMake、openssl、ncurses依赖包（在root用户下安装）
####a、安装CMake依赖包
![1.jpg-29.9kB][15]
```linux
[root@pseudo opt]# yum install cmake
```
####b、安装openssl依赖包
![1.jpg-33.4kB][16]
```linux
[root@pseudo opt]# yum install openssl-devel
```
####c、安装ncurses依赖包
![1.jpg-34.9kB][17]
```linux
[root@pseudo opt]# yum install ncurses-devel
```
###6) 添加配置镜像，路径：/opt/modules/apache-maven-3.0.5/conf/settings.xml(可以不需要配置)
#### a.在<mirrors></mirros>里添加
![1.jpg-27.5kB][18]
```xml
<mirror>
      <id>nexus-osc</id>
      <mirrorOf>*</mirrorOf>
      <name>Nexus osc</name>
      <url>http://maven.oschina.net/content/groups/public/</url>
</mirror>
```
#### b.在<profiles></profiles>里添加
```linux
<profile>
   <id>jdk-1.7</id>
   <activation>
	 <jdk>1.7</jdk>
   </activation>
   <repositories>
	 <repository>
	   <id>nexus</id>
	   <name>local private nexus</name>
	   <url>http://maven.oschina.net/content/groups/public/</url>
	   <releases>
		 <enabled>true</enabled>
	   </releases>
	   <snapshots>
		 <enabled>false</enabled>
	   </snapshots>
	 </repository>
   </repositories>
   <pluginRepositories>
	 <pluginRepository>
	   <id>nexus</id>
	  <name>local private nexus</name>
	   <url>http://maven.oschina.net/content/groups/public/</url>
	   <releases>
		 <enabled>true</enabled>
	   </releases>
	   <snapshots>
		 <enabled>false</enabled>
	   </snapshots>
	 </pluginRepository>
   </pluginRepositories>
 </profile>
```
###7) 解压Hadoop源码包并编译Hadoop 2.x源码
####a、解压Hadoop源码包
![1.jpg-24.5kB][19]
```linux
[hadoop@pseudo hadoop]$ tar -zxf /opt/softwares/hadoop-2.5.0-src.tar.gz -C /opt/modules/hadoop
[hadoop@pseudo hadoop]$ ls /opt/modules/hadoop
hadoop-2.5.0  hadoop-2.5.0-src
```
####b、编译Hadoop 2.x源码
![1.jpg-68.3kB][20]
```linux
[hadoop@pseudo hadoop-2.5.0-src]$ mvn package -DskipTests -Pdist,native
```
####c、编译命令行成功代码
![1.jpg-34kB][21]
###8) 替换hadoop 2.x下的native目录
![1.jpg-34.9kB][22]
```linxu
[hadoop@pseudo native]$ cd /opt/modules/hadoop/hadoop-2.5.0-src/hadoop-dist/target/hadoop-2.5.0/lib
[hadoop@pseudo lib]$ cp -r native/ /opt/modules/hadoop/hadoop-2.5.0/lib/
```
###9) 测试Hadoop 2.x是否编译成功
![1.jpg-36.1kB][23]


  [1]: http://static.zybuluo.com/myouhao/owgsw51hneusox00u389li4z/1.jpg
  [2]: http://static.zybuluo.com/myouhao/uf2xtjp6fjqf393ats5oqlbu/1.jpg
  [3]: http://static.zybuluo.com/myouhao/r4vuhq91jht12mml58b87mzr/1.jpg
  [4]: http://static.zybuluo.com/myouhao/enxkg101onpas0ek0z4rwa7e/1.jpg
  [5]: http://static.zybuluo.com/myouhao/q31wz1c0w5b8m58frpxbz0pp/1.jpg
  [6]: http://static.zybuluo.com/myouhao/q17gc1b4wxey65u761h64sn0/1.jpg
  [7]: http://static.zybuluo.com/myouhao/g0ag1l254o4non8xykrntpnu/1.jpg
  [8]: http://static.zybuluo.com/myouhao/rmvk7popuix2epta5ujnkohx/1.jpg
  [9]: http://static.zybuluo.com/myouhao/elbwklior40iw74alprmt806/1.jpg
  [10]: http://static.zybuluo.com/myouhao/kkq761t36foieq1uqtuoveov/1.jpg
  [11]: http://static.zybuluo.com/myouhao/eg2w0p5od5gefljs89ubgejv/1.jpg
  [12]: http://static.zybuluo.com/myouhao/kzqqf6c0d7zdqya59qvkepho/1.jpg
  [13]: http://static.zybuluo.com/myouhao/d0eikgd33q7pxpoaf35w670r/1.jpg
  [14]: http://static.zybuluo.com/myouhao/7h8iuqp7eg5v6lm0znyzqbj8/1.jpg
  [15]: http://static.zybuluo.com/myouhao/xuk2npfz4idc2q62z1e5kqq1/1.jpg
  [16]: http://static.zybuluo.com/myouhao/100tjtiqupqd79ofuxruekmf/1.jpg
  [17]: http://static.zybuluo.com/myouhao/prncs90he4q99cgntn05g3zl/1.jpg
  [18]: http://static.zybuluo.com/myouhao/yms420ukf9rx592kosuuzttj/1.jpg
  [19]: http://static.zybuluo.com/myouhao/kf5u3dvpgz90lyc54df1akeh/1.jpg
  [20]: http://static.zybuluo.com/myouhao/jbna89fh4clfvl9oj9hig1bn/1.jpg
  [21]: http://static.zybuluo.com/myouhao/8zvb8hgzi9i8911zg1qz3vq4/1.jpg
  [22]: http://static.zybuluo.com/myouhao/xpe0e36kn8w41641e7tcpgro/1.jpg
  [23]: http://static.zybuluo.com/myouhao/lfwql5hjv6ci3hev9w07g6nr/1.jpg