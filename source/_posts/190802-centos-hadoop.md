---
layout: post
title: 虚拟机Linux下 hadoop 环境搭建准备
date: 2019-08-02 08:58:20
comments: true
tags: 
  - linux
  - hadoop
---
### 主机名修改及映射
#### a. 修改当时生效
```linux
hostname heyouhao
```
#### b. 修改后重启生效
```linux
vi /etc/sysconfig/network
```
<!--more-->
#### c. 主机名映射
```linux
vi /etc/hosts
127.0.0.1   pseudoStudy
```

### 克隆linux系统注意
#### a. 在/etc/udev/rules.d/70-persistent-net.rules文件里把原eth0的相关信息删除，将eth1的name改为eth0
```linux
[hadoop@pseudo /]$ sudo vi /etc/udev/rules.d/70-persistent-net.rules
```
#### b. 将/etc/sysconfig/network-scripts/ifcfg-eth0 里面的mac地址改为上面eth0的mac地址
```linux
[hadoop@pseudo /]$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
#### c. 重启
```linux
[hadoop@pseudo /]$ sudo reboot
```
### ip地址修改
#### a. 进入vi设置ip地址
```linux
vi /etc/sysconfig/network-scripts/ifcfg-eth0 
```
#### b. 修改内容
```linux
BOOTPROTO="static"   #dhcp改为static  
ONBOOT="yes"           #开机启用本配置  
IPADDR=192.168.1.81   #静态IP  
GATEWAY=192.168.1.2    #默认网关  
NETMASK=255.255.255.0  #子网掩码  
DNS1=210.21.4.130       #DNS 配置 
IPV4_FAILURE_FATAL=yes
IPV6INIT=no
NAME="System eth0"
```

#### c. 重启网卡生效
```linux
service network restart
```
### jdk卸载安装
#### a. 查看jdk默认安装情况
```linux
[root@pseudoStudy ~]# java -version
java version "1.7.0_09-icedtea"
OpenJDK Runtime Environment (rhel-2.3.4.1.el6_3-x86_64)
OpenJDK 64-Bit Server VM (build 23.2-b09, mixed mode)
```
#### b. 卸载自带jdk
```
[root@pseudoStudy ~]# rpm -qa|grep java               //查看
java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64
tzdata-java-2012j-1.el6.noarch
java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64
[root@pseudoStudy ~]# rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.x86_64   //卸载
[root@pseudoStudy ~]# rpm -e --nodeps tzdata-java-2012j-1.el6.noarch java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.x86_64      //卸载
```
#### c. 新建softwares目录，将jdk安装压缩包到softwares目录下
```
[hadoop@fyd01 opt]$ sudo mkdir softwares
```
#### d. 解压安装
#### e. 配置JAVA_HOME环境变量
```
[root@pseudoStudy jdk1.7.0_67]# pwd
/opt/modules/jdk1.7.0_67
[root@pseudoStudy jdk1.7.0_67]# vim /etc/profile
```
添加内容
```
export JAVA_HOME=/opt/modules/jdk1.7.0_67
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
刷新测试
```
[root@pseudoStudy jdk1.7.0_67]# source /etc/profile
[root@pseudoStudy jdk1.7.0_67]# java -version
java version "1.7.0_67"
Java(TM) SE Runtime Environment (build 1.7.0_67-b01)
Java HotSpot(TM) 64-Bit Server VM (build 24.65-b04, mixed mode)
```
### Hadoop准备
#### a. 创建hadoop用户
```
[root@pseudoStudy jdk1.7.0_67]# useradd hadoop
[root@pseudoStudy jdk1.7.0_67]# passwd hadoop
```
#### b. 给hadoop用户设置sudo权限
```
[root@pseudoStudy jdk1.7.0_67]# ll /etc/sudoers
-r--r-----. 1 root root 4002 Mar  2  2012 /etc/sudoers
[root@pseudoStudy jdk1.7.0_67]# chmod 640 /etc/sudoers
[root@pseudoStudy jdk1.7.0_67]# ll /etc/sudoers
-rw-r-----. 1 root root 4002 Mar  2  2012 /etc/sudoers
[root@pseudoStudy jdk1.7.0_67]# vi /etc/sudoers
```
添加内容
```
hadoop  ALL=(root)NOPASSWD:ALL
```
#### c. 创建目录，进入/opt/modules 目录，创建 hadoop子目录
```
[hadoop@fyd01 opt]$ sudo mkdir modules
[hadoop@fyd01 opt]$ sudo mkdir tools
[hadoop@fyd01 opt]$ sudo mkdir datas
[hadoop@fyd01 opt]$ ls
datas  modules  softwares  tools
[hadoop@fyd01 modules]# sudo mkdir hadoop
[hadoop@fyd01 modules]# sudo chown -R hadoop:hadoop hadoop/
```
#### d. 解压hadoop文件
```
[hadoop@fyd01 softwares]# tar -zxvf hadoop-2.5.0.tar.gz  -C /opt/modules/hadoop/
```

#### e.防火墙关闭
```linux
service iptables status   //查看防火墙状态
service iptables stop
chkconfig iptables off    //启动关闭防火墙
```