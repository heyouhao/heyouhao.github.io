---
layout: post
title: centOS 硬件信息查看
date: 2019-08-02 08:55:20
comments: true
tags: 
  - linux
---

## 1. 系统
#### 1.1 版本

uname -a 能确认是64位还是32位，其它的信息不多

```linux
[root@localhost ~]# uname -a
Linux localhost.localdomain 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
```

more /etc/*release 可以看到更多信息
<!--more-->
```linux
[root@heyouhao ~]# more /etc/*release 
::::::::::::::
/etc/centos-release
::::::::::::::
CentOS Linux release 7.4.1708 (Core) 
::::::::::::::
/etc/os-release
::::::::::::::
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

::::::::::::::
/etc/redhat-release
::::::::::::::
CentOS Linux release 7.4.1708 (Core) 
::::::::::::::
/etc/system-release
::::::::::::::
CentOS Linux release 7.4.1708 (Core)
```
### 1.2 核数
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

```linux
[root@heyouhao ~]# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
      2  Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.50GHz
```

cat /proc/cpuinfo | grep physical | uniq -c

```linux
[root@heyouhao ~]# cat /proc/cpuinfo | grep physical | uniq -c
      1 physical id	: 0
      1 address sizes	: 46 bits physical, 48 bits virtual
      1 physical id	: 0
      1 address sizes	: 46 bits physical, 48 bits virtual

```
 由2个1核的CPU组成2核
### 1.3 运行模式
getconf LONG_BIT  CPU运行在多少位模式下

```linux
[root@heyouhao ~]# getconf LONG_BIT
64
```

如果是32，说明当前CPU运行在32bit模式下, 但不代表CPU不支持64bit
cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l 是否支持64位

```linux
[root@heyouhao ~]# cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l 
2
```

结果大于0, 说明支持64bit计算. lm指long mode, 支持lm则是64bit
### 1.4 计算机名
hostname

```linux
[root@heyouhao ~]# hostname
heyouhao
```

### 1.5 查看环境变量
env

```linux

[root@heyouhao ~]# env
XDG_SESSION_ID=10224
HOSTNAME=heyouhao
TERM=xterm
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=58.252.60.157 13291 22
SSH_TTY=/dev/pts/1
JRE_HOME=/usr/local/java/jdk1.8.0_152/jre
MAIL=/var/spool/mail/root
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/java/jdk1.8.0_152/bin:/usr/local/java/jdk1.8.0_152/jre/bin:/opt/gitlab/embedded/sbin/nginx:/root/bin
```

### 1.6 系统允许多长时间了/负载数
uptime

```linux
[root@heyouhao ~]# uptime
 10:41:27 up 60 days, 10:12,  1 user,  load average: 0.00, 0.01, 0.05
```

1.当前时间10:41:27
2.系统运行了多少时间,60days 10:12（60天10小时12分)
3.多少个用户,1 users
4.平均负载：0.00, 0.01, 0.05，最近1分钟、5分钟、15分钟系统的负载

直接查看平均负载情况 cat /proc/loadavg

```linux
[root@heyouhao ~]# cat /proc/loadavg
0.08 0.04 0.05 1/414 11179
```

除了前3个数字表示平均进程数量外，后面的1个分数，分母表示系统进程总数，分子表示正在运行的进程数；最后一个数字表示最近运行的进程ID
## 2. 资源
### 2.1 内存
 cat /proc/meminfo 内存的详细信息

```linux
[root@heyouhao ~]#  cat /proc/meminfo
MemTotal:        3881692 kB
MemFree:          126160 kB
MemAvailable:     265320 kB
Buffers:           79860 kB
Cached:           272776 kB
SwapCached:        27088 kB
Active:          2620216 kB
```

MemTotal总内存，MemFree可用内存
free -m(-m，单位是m，如果是-g，单位是g）查看可用内存

```linux
[root@heyouhao ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3790        3250         116          32         423         256
Swap:          2047         325        1722
```

空闲内存total-used=free+buff/cache
我们通过free命令查看机器空闲内存时，会发现free的值很小。这主要是因为，在linux中有这么一种思想，内存不用白不用，因此它尽可能的cache和buffer一些数据，以方便下次使用。但实际上这些内存也是可以立刻拿来使用的。
## 3. 磁盘和分区
###  3.1 查看各分区使用情况
df -h

```linux
[root@heyouhao ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G  8.2G   30G  22% /
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G  4.0K  1.9G   1% /dev/shm
tmpfs           1.9G  360K  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
tmpfs           380M     0  380M   0% /run/user/0
```

### 3.2 查看指定目录的大小
du -sh <目录名>

```linux
[root@heyouhao local]# du -sh tomcat/
220M	tomcat/
```

### 3.3 查看所有分区
fdisk -l

```linux
[root@heyouhao local]#  fdisk -l

Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0008d73a

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83884031    41940992   83  Linux
```

### 3.4 查看所有交换分区
swapon -s

```linux
[root@heyouhao local]# swapon -s
Filename				Type		Size	Used	Priority
/mnt/swap                              	file	2097148	333604	-1
```
## 4. 网络

### 4.1 查看所有网络接口的属性
ifconfig

```linux
[root@heyouhao local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.56.232  netmask 255.255.240.0  broadcast 172.18.63.255
        ether 00:16:3e:04:4a:da  txqueuelen 1000  (Ethernet)
        RX packets 2846405  bytes 626223429 (597.2 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3581982  bytes 1494945222 (1.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1  (Local Loopback)
        RX packets 51065350  bytes 47522440520 (44.2 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 51065350  bytes 47522440520 (44.2 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 4.2 带宽
ethtool 网卡名

```linux
[root@localhost ~]# ethtool eth0
Settings for eth0:
	Supported ports: [ TP ]
	Supported link modes:   1000baseT/Full 
	                        10000baseT/Full 
	Supported pause frame use: No
	Supports auto-negotiation: No
	Advertised link modes:  Not reported
	Advertised pause frame use: No
	Advertised auto-negotiation: No
	Speed: 10000Mb/s
	Duplex: Full
	Port: Twisted Pair
	PHYAD: 0
	Transceiver: internal
	Auto-negotiation: off
	MDI-X: Unknown
	Supports Wake-on: uag
	Wake-on: d
	Link detected: yes
```

看Speed
### 4.3 查看路由表
route -n

```linux
[root@localhost ~]#  route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.172.19.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
0.0.0.0         192.172.19.254  0.0.0.0         UG    0      0        0 eth0
```

### 4.4 查看所有监听端口
netstat -lntp

```linux
[root@heyouhao local]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      530/node_exporter   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      864/unicorn master  
tcp        0      0 127.0.0.1:9168          0.0.0.0:*               LISTEN      522/ruby            
```

### 4.5 查看所有已经建立的连接
netstat -antp

```linux
[root@heyouhao local]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      530/node_exporter   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      864/unicorn master  
tcp        0      0 127.0.0.1:9168          0.0.0.0:*               LISTEN      522/ruby            
```

### 4.6 某端口使用情况
lsof -i:端口号

```linux
没有安装
```
或者
netstat -apn|grep 端口号

```linux
[root@heyouhao local]# netstat -apn|grep 8081
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      10575/java
```

## 5. 进程
###   5.1 查看所有进程
ps -ef,使用ps -ef|gerp tomcat过滤

```linux
[root@heyouhao local]# ps -ef|grep tomcat
root     10575     1  0 Aug07 ?        01:27:11 /usr/local/java/jdk1.8.0_152/jre/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/apache-tomcat-8.5.24/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
root     13621  6024  0 11:06 pts/1    00:00:00 grep --color=auto tomcat
```

ps -aux可以看到进程占用CPU,内存情况

```linux
[root@heyouhao local]# ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  43264  2180 ?        Ss   Aug01   0:31 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    Aug01   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Aug01   0:07 [ksoftirqd/0]
```

### 5.2 实时显示进程状态
top

```linux
[root@heyouhao local]# top
top - 11:09:02 up 60 days, 10:40,  1 user,  load average: 0.00, 0.01, 0.05
Tasks: 180 total,   1 running, 179 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.2 us,  0.2 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  3881692 total,   111144 free,  3331120 used,   439428 buff/cache
KiB Swap:  2097148 total,  1763544 free,   333604 used.   246192 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                    
  534 gitlab-+  20   0  730620 109304   4960 S   1.7  2.8 586:31.62 prometheus                                                                                                                                                 
  535 git       20   0  858816 529380   1516 S   0.7 13.6 598:38.91 bundle                                                                                                                                                                                                                                                                                                                                                                                                                                                   
13808 root      20   0  157724   2284   1548 R   0.3  0.1   0:00.02 top                                                                                                                                                        
    1 root      20   0   43264   2180   1304 S   0.0  0.1   0:31.03 systemd                                                                                                                                                          
```

## 6. 用户
### 6.1 查看活动用户
w

```linux
[root@localhost ~]# w
 11:10:50 up 23:03,  5 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.172.19.206   08:03   40:33  60.88s  0.10s -bash
root     pts/2    192.172.19.206   08:03    1:56m  0.06s  0.00s tail -f retail_error.log
root     pts/3    192.172.31.53    08:22    2:33m  0.10s  0.10s -bash
```

### 6.2 查看指定用户的信息
id <用户名>

```linux
[root@localhost ~]# id root
uid=0(root) gid=0(root) 组=0(root)
```

### 6.3 查看用户登录日志
last

```linux
uid=0(root) gid=0(root) 组=0(root)
[root@localhost ~]# last 
root     pts/3        192.172.31.53    Sun Sep 30 08:22   still logged in   
root     pts/2        192.172.19.206   Sun Sep 30 08:03   still logged in      
root     pts/1        192.172.31.53    Sat Sep 29 13:49 - 14:04  (00:14)    
root     pts/0        192.172.56.3     Sat Sep 29 11:59 - 15:22  (03:22)    
reboot   system boot  2.6.32-431.el6.x Sat Sep 29 11:57 - 11:11  (23:13)   
```

### 6.4 查看系统所有用户
cut -d: -f1 /etc/passwd

```linux
[root@localhost ~]# 
[root@localhost ~]# cut -d: -f1 /etc/passwd
root
bin
daemon
adm
```

```linux
参考：https://blog.csdn.net/dream_broken/article/details/52883883
```
