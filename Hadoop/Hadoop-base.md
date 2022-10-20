<!-- TOC -->

- [hadoop](#hadoop)
  - [生态体系](#生态体系)
  - [集群安装](#集群安装)
    - [基础环境](#基础环境)
      - [前期ip网络、host配置](#前期ip网络host配置)
      - [设置软件库](#设置软件库)
      - [配置用户【atguigu】](#配置用户atguigu)
      - [归属配置](#归属配置)
      - [卸载虚拟机自带的 jdk](#卸载虚拟机自带的-jdk)
    - [hadoop安装](#hadoop安装)
    - [克隆节点](#克隆节点)
  - [运行模式及准备](#运行模式及准备)
    - [运行模式](#运行模式)
    - [免密登录设置](#免密登录设置)
    - [脚本下发](#脚本下发)
  - [配置集群](#配置集群)
    - [集群基础](#集群基础)
    - [配置【core-site.xml】](#配置core-sitexml)
    - [配置【hdfs-site.xml】](#配置hdfs-sitexml)
    - [配置【yarn-site.xml】](#配置yarn-sitexml)
    - [配置【mapred-site.xml】](#配置mapred-sitexml)
    - [分发配置到各集群](#分发配置到各集群)
    - [群起集群](#群起集群)
      - [配置works](#配置works)
      - [初始化NameNode](#初始化namenode)
      - [启动hdfs](#启动hdfs)
  - [集群应用](#集群应用)
    - [集群测试](#集群测试)
      - [配置历史服务器](#配置历史服务器)
      - [日志收集](#日志收集)
      - [集群重新初始化](#集群重新初始化)
    - [集群的常用操作](#集群的常用操作)
      - [启停](#启停)
      - [启停脚本](#启停脚本)
      - [jps脚本](#jps脚本)
      - [集群常用端口](#集群常用端口)
      - [时间同步](#时间同步)

<!-- /TOC -->

<a id="markdown-hadoop" name="hadoop"></a>
# hadoop

<a id="markdown-生态体系" name="生态体系"></a>
## 生态体系

![](hadoop生态体系.png)


<a id="markdown-集群安装" name="集群安装"></a>
## 集群安装

<a id="markdown-基础环境" name="基础环境"></a>
### 基础环境

<a id="markdown-前期ip网络host配置" name="前期ip网络host配置"></a>
#### 前期ip网络、host配置

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=64bc05ba-09a0-44a1-9bb0-41af7cb70272
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.217.102
NETMASK=255.255.255.0
GATEWAY=192.168.217.2
DNS1=192.168.217.2
DNS2=114.114.114.114

```

```shell
# 修改主机名称 
hostnamectl set-hostname hadoop102
hostname hadoop102

# 修改host名称
vi /etc/hosts

127.0.0.1   loc

192.168.217.102 hadoop102
192.168.217.103 hadoop103
192.168.217.104 hadoop104

```

<a id="markdown-设置软件库" name="设置软件库"></a>
#### 设置软件库

```shell
# 针对rhel配置软件库：
yum install -y epel-release

# 关闭防火墙
systemctl stop firewalld
# 关闭开机启动
systemctl disable firewalld.service

```

<a id="markdown-配置用户atguigu" name="配置用户atguigu"></a>
#### 配置用户【atguigu】

```shell
# 创建atguigu用户，并修改atguigu用户的密码
useradd atguigu
passwd atguigu

# 配置atguigu用户具有root权限，方便后期加sudo执行root权限的命令
vi /etc/sudoers

```

在%wheel这行下面添加一行 

```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
atguigu   ALL=(ALL)     NOPASSWD:ALL

```

注意：atguigu这一行不要直接放到root行下面，

因为所有用户都属于wheel组，你先配置了atguigu具有免密功能，

但是程序执行到%wheel行时，该功能又被覆盖回需要密码。

所以atguigu要放到%wheel这行下面。

<a id="markdown-归属配置" name="归属配置"></a>
#### 归属配置

```shell
# 存放各软件包
mkdir /opt/software
# 存放各软件运行环境
mkdir /opt/module

# 修改module、software文件夹的所有者和所属组均为atguigu用户
chown atguigu:atguigu /opt/module
chown atguigu:atguigu /opt/software
```

<a id="markdown-卸载虚拟机自带的-jdk" name="卸载虚拟机自带的-jdk"></a>
#### 卸载虚拟机自带的 jdk 

```shell
# rpm -qa 查询rpm软件包
# grep -i 忽略大小写
# xargs -n1 每次只传递一个参数
# rpm -e -nodeps 强制卸载
rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps
```

jdk安装：

```shell
# 解压至指定目录
tar -zxvf jdk-8u202-linux-x64.tar.gz -C /opt/module

# 配置环境变量
vi /etc/profile.d/my_env.sh
```

【my_env.sh】编辑内容：

```shell
# JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_202
export PATH=$PATH:$JAVA_HOME/bin
```

```shell
# 生效环境变量
source /etc/profile

# 测试java
java -version
```

<a id="markdown-hadoop安装" name="hadoop安装"></a>
### hadoop安装

下载地址：https://archive.apache.org/dist/hadoop/common/hadoop-3.1.3/

下载安装包【hadoop-3.1.3.tar.gz】

```shell
# 解压hadoop压缩包
tar -zxvf /opt/software/hadoop-3.1.3.tar.gz -C /opt/module

# pwd
/opt/module/hadoop-3.1.3

# vi /etc/profile.d/my_env.sh
```

添加以下配置：

```shell
# HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```


```shell
# 使文件生效
source /etc/profile

# hadoop version
Hadoop 3.1.3
Source code repository https://gitbox.apache.org/repos/asf/hadoop.git -r ba631c436b806728f8ec2f54ab1e289526c90579
Compiled by ztang on 2019-09-12T02:47Z
Compiled with protoc 2.5.0
From source with checksum ec785077c385118ac91aadde5ec9799
This command was run using /opt/module/hadoop-3.1.3/share/hadoop/common/hadoop-common-3.1.3.jar

```

hadoop 重要目录：

1. bin目录：存放对Hadoop相关服务（hdfs，yarn，mapred）进行操作的脚本
2. etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
3. lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
4. sbin目录：存放启动或停止Hadoop相关服务的脚本
5. share目录：存放Hadoop的依赖jar包、文档、和官方案例

<a id="markdown-克隆节点" name="克隆节点"></a>
### 克隆节点

利用刚配置好的 hadoop102 ，克隆生成 hadoop103 hadoop104

并修改ip地址和主机名称。


<a id="markdown-运行模式及准备" name="运行模式及准备"></a>
## 运行模式及准备

<a id="markdown-运行模式" name="运行模式"></a>
### 运行模式

Hadoop运行模式包括：

- 本地模式
- 伪分布式模式
- 完全分布式模式

<a id="markdown-免密登录设置" name="免密登录设置"></a>
### 免密登录设置

```shell
# 远程登录 hadoop103 机器，会生成 ~/.ssh 隐藏文件夹
ssh hadoop103

cd ~/.ssh

# 生成公钥和密钥 默认参数（直接回车即可）
ssh-keygen -t rsa

# 拷贝公钥到免密登录的机器
ssh-copy-id hadoop102 # 本机也要拷贝，否则hadoop启动namenode报无权限
ssh-copy-id hadoop103
ssh-copy-id hadoop104

```

集群内各节点最好都设置下ssh免密登录，以免后期从 hadoop103 登录到 hadoop102 时无权限

<a id="markdown-脚本下发" name="脚本下发"></a>
### 脚本下发

```shell
vi /bin/xsync

# 内容如下：
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done

```

```shell
# 赋予脚本权限
chmod 775 /bin/xsync
```

测试使用下发脚本拷贝文件至集群其他机器：

```shell
cd /opt/software

# 测试文件同步至其他节点同路径
xsync /opt/software/test_xia_fa
```

<a id="markdown-配置集群" name="配置集群"></a>
## 配置集群

<a id="markdown-集群基础" name="集群基础"></a>
### 集群基础

--   | hadoop102 | hadoop103 | hadoop104
-|-----------|-----------|----------
HDFS | NameNode/DataNode | DataNode | SecondaryNameNode/DataNode
YARN | NodeManager | ResourceManager/NodeManager | NodeManager

默认配置文件：

要获取的默认文件 | 文件存放在Hadoop的jar包中的位置
---------|---------------------
[core-default.xml] | hadoop-common-3.1.3.jar/core-default.xml
[hdfs-default.xml] | hadoop-hdfs-3.1.3.jar/hdfs-default.xml
[yarn-default.xml] | hadoop-yarn-common-3.1.3.jar/yarn-default.xml
[mapred-default.xml] | hadoop-mapreduce-client-core-3.1.3.jar/mapred-default.xml

自定义配置：

core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml

四个配置文件存放在`$HADOOP_HOME/etc/hadoop`这个路径上，用户可以根据项目需求重新进行修改配置。

<a id="markdown-配置core-sitexml" name="配置core-sitexml"></a>
### 配置【core-site.xml】

```shell
cd $HADOOP_HOME/etc/hadoop

vi core-site.xml
```

配置内容如下：

```xml
<configuration>
    <!-- 指定NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop102:8020</value>
    </property>

    <!-- 指定hadoop数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
    </property>

    <!-- 配置HDFS网页登录使用的静态用户为 atguigu -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>atguigu</value>
    </property>
</configuration>

```

<a id="markdown-配置hdfs-sitexml" name="配置hdfs-sitexml"></a>
### 配置【hdfs-site.xml】

```shell
cd $HADOOP_HOME/etc/hadoop

vi hdfs-site.xml
```

配置内容如下：

```xml
<configuration>
	<!-- nn web端访问地址-->
	<property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop102:9870</value>
    </property>
	<!-- 2nn web端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop104:9868</value>
    </property>
</configuration>
```


<a id="markdown-配置yarn-sitexml" name="配置yarn-sitexml"></a>
### 配置【yarn-site.xml】

```shell
cd $HADOOP_HOME/etc/hadoop

vi yarn-site.xml
```

配置内容如下：

```xml
<configuration>
    <!-- 指定MR走shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- 指定ResourceManager的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
    </property>

    <!-- 环境变量的继承 hadoop3.2+版本不需要此配置 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>

</configuration>
```

<a id="markdown-配置mapred-sitexml" name="配置mapred-sitexml"></a>
### 配置【mapred-site.xml】

```shell
cd $HADOOP_HOME/etc/hadoop

vi mapred-site.xml
```

配置内容如下：

```xml
<configuration>

	<!-- 指定MapReduce程序运行在Yarn上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

</configuration>
```

<a id="markdown-分发配置到各集群" name="分发配置到各集群"></a>
### 分发配置到各集群

```shell
[root@hadoop102 hadoop]# xsync /opt/module/hadoop-3.1.3/etc/hadoop/
==================== hadoop103 ====================
sending incremental file list
hadoop/
hadoop/core-site.xml
hadoop/hdfs-site.xml
hadoop/mapred-site.xml
hadoop/yarn-site.xml

sent 3,513 bytes  received 139 bytes  2,434.67 bytes/sec
total size is 107,770  speedup is 29.51
==================== hadoop104 ====================
sending incremental file list
hadoop/
hadoop/core-site.xml
hadoop/hdfs-site.xml
hadoop/mapred-site.xml
hadoop/yarn-site.xml

sent 3,513 bytes  received 139 bytes  7,304.00 bytes/sec
total size is 107,770  speedup is 29.51

```

<a id="markdown-群起集群" name="群起集群"></a>
### 群起集群

<a id="markdown-配置works" name="配置works"></a>
#### 配置works

```shell
vi /opt/module/hadoop-3.1.3/etc/hadoop/workers
```

内容如下：

```shell
hadoop102
hadoop103
hadoop104
```

注意：该文件中添加的内容结尾不允许有空格，文件中不允许有空行。

```shell
# 同步所有配置
xsync /opt/module/hadoop-3.1.3/etc/hadoop/

==================== hadoop103 ====================
sending incremental file list
hadoop/
hadoop/workers

sent 982 bytes  received 46 bytes  2,056.00 bytes/sec
total size is 107,790  speedup is 104.85
==================== hadoop104 ====================
sending incremental file list
hadoop/
hadoop/workers

sent 982 bytes  received 46 bytes  2,056.00 bytes/sec
total size is 107,790  speedup is 104.85

```


<a id="markdown-初始化namenode" name="初始化namenode"></a>
#### 初始化NameNode

```shell
[root@hadoop102 hadoop-3.1.3]# pwd
/opt/module/hadoop-3.1.3
# 首次格式化 NameNode
[root@hadoop102 hadoop-3.1.3]# hdfs namenode -format
..........

```

注意：格式化NameNode，会产生新的集群id，导致NameNode和DataNode的集群id不一致，集群找不到已往数据。

如果集群在运行过程中报错，需要重新格式化NameNode的话，一定要先停止namenode和datanode进程，

并且要删除所有机器的data和logs目录，然后再进行格式化。

<a id="markdown-启动hdfs" name="启动hdfs"></a>
#### 启动hdfs

如果使用的是root用户操作，则需要补充以下环境变量，建议在专门用户下进行操作。

```shell
vi /etc/profile.d/my_env.sh

# 新增内容如下
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root

```

在各节点通过 jps 查看节点信息

```shell
# 启动集群
[root@hadoop102 hadoop-3.1.3]# sbin/start-dfs.sh


[root@hadoop102 ~]# jps
36450 NameNode
1443 Jps
36639 DataNode

[root@hadoop103 ~]# jps
2500 Jps
29893 DataNode

[root@hadoop104 ~]# jps
26982 DataNode
27110 SecondaryNameNode
130774 Jps

# 在配置了ResourceManager的节点（hadoop103）启动YARN
[root@hadoop103 hadoop-3.1.3]# cd /opt/module/hadoop-3.1.3/
[root@hadoop103 hadoop-3.1.3]# sbin/start-yarn.sh
```

Web端查看HDFS的 NameNode
- 浏览器中输入： http://hadoop102:9870 或者 http://192.168.217.102:9870/
- 查看HDFS上存储的数据信息

Web端查看YARN的 ResourceManager
- 浏览器中输入： http://hadoop103:8088 或者 http://192.168.217.103:8088/
- 查看YARN上运行的Job信息


<a id="markdown-集群应用" name="集群应用"></a>
## 集群应用

<a id="markdown-集群测试" name="集群测试"></a>
### 集群测试

```shell
# 根目录下创建 input 通过 NameNode 地址查看新增的 input 文件夹
hadoop fs -mkdir /input

# 执行测试程序
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output

```

<a id="markdown-配置历史服务器" name="配置历史服务器"></a>
#### 配置历史服务器

```shell
cd /opt/module/hadoop-3.1.3/etc/hadoop
vi mapred-site.xml
```

内容如下：

```shell

<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop102:10020</value>
</property>

<!-- 历史服务器web端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>

```

下发配置：

```shell
xsync $HADOOP_HOME/etc/hadoop/mapred-site.xml

# 启动历史服务器
[atguigu@hadoop102 hadoop-3.1.3]$ mapred --daemon start historyserver
[atguigu@hadoop102 hadoop-3.1.3]$ jps
29569 DataNode
63857 Jps
30997 NodeManager
63708 JobHistoryServer
29358 NameNode

```

<a id="markdown-日志收集" name="日志收集"></a>
#### 日志收集

```shell
[atguigu@hadoop102 ~]$ cd $HADOOP_HOME/etc/hadoop/

[atguigu@hadoop102 hadoop]$ vi yarn-site.xml

```

配置内容如下：

```xml
    <!-- 开启日志聚集功能 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- 设置日志聚集服务器地址 -->
    <property>  
        <name>yarn.log.server.url</name>  
        <value>http://hadoop102:19888/jobhistory/logs</value>
    </property>
    <!-- 设置日志保留时间为7天 -->
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
```

分发配置，重启yarn，启动测试

```shell
[atguigu@hadoop102 hadoop]$ xsync $HADOOP_HOME/etc/hadoop/yarn-site.xml
[atguigu@hadoop102 hadoop]$ cd $HADOOP_HOME

# 停掉 102上 historyserver，等yarn重启后再启动 historyserver 
[atguigu@hadoop102 hadoop-3.1.3]$ mapred --daemon stop historyserver

# 重启103上的 yarn
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/stop-yarn.sh
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh

# 启动 102 上的 historyserver
[atguigu@hadoop102 hadoop-3.1.3]$ mapred --daemon start historyserver

# 执行测试程序
[atguigu@hadoop102 hadoop-3.1.3]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /output2
```

<a id="markdown-集群重新初始化" name="集群重新初始化"></a>
#### 集群重新初始化

当遇到一些问题时，可以选择重新初始化节点

```shell
[atguigu@hadoop102 ~]$ cd /opt/module/hadoop-3.1.3
# 停掉进程 个别进程也可以通过 kill -9 pid 方式杀死
[atguigu@hadoop102 hadoop-3.1.3]$ sbin/stop-dfs.sh

[atguigu@hadoop103 ~]$ cd /opt/module/hadoop-3.1.3
# 停掉进程 个别进程也可以通过 kill -9 pid 方式杀死
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/stop-yarn.sh

# 删除所有节点的 data 和 logs 文件夹，然后进行初始化
[atguigu@hadoop102 hadoop-3.1.3]$ rm -rf data/ logs/
[atguigu@hadoop103 hadoop-3.1.3]$ rm -rf data/ logs/
[atguigu@hadoop104 hadoop-3.1.3]$ rm -rf data/ logs/

# 重新初始 namenode 信息
[root@hadoop102 hadoop-3.1.3]# hdfs namenode -format

```

<a id="markdown-集群的常用操作" name="集群的常用操作"></a>
### 集群的常用操作

<a id="markdown-启停" name="启停"></a>
#### 启停

```shell
# 启动、停止 hdfs
start-dfs.sh/stop-dfs.sh

# 启动、停止 yarn
start-yarn.sh/stop-yarn.sh

# 分别启动 hdfs 组件
hdfs --daemon start/stop namenode/datanode/secondarynamenode

# 启动停止 yarn
yarn --daemon start/stop  resourcemanager/nodemanager
```

<a id="markdown-启停脚本" name="启停脚本"></a>
#### 启停脚本

```shell
cd /home/atguigu/bin
vi myhadoop.sh
```

```shell
#!/bin/bash

if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi

case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac

```

<a id="markdown-jps脚本" name="jps脚本"></a>
#### jps脚本

```shell
cd /home/atguigu/bin
vi jpsall
```

```shell
#!/bin/bash

for host in hadoop102 hadoop103 hadoop104
do
        echo =============== $host ===============
        ssh $host jps 
done

```


<a id="markdown-集群常用端口" name="集群常用端口"></a>
#### 集群常用端口

hadoop3.x 常用端口

- NameNode内部常用端口:8020/9000/9820
- NameNode用户查询端口:9870
- Yarn查看任务运行:8088
- 历史服务器:19888

<a id="markdown-时间同步" name="时间同步"></a>
#### 时间同步

**主要节点配置**

修改【/etc/ntp.conf】

```shell
# 切换用户到 root
[atguigu@hadoop102 ~]$ su root
...
[root@hadoop102 ~]$ su root

# 查看时间服务器 ntpd 服务情况
[root@hadoop102 ~]# systemctl status ntpd
[root@hadoop102 ~]# systemctl start ntpd
[root@hadoop102 ~]# systemctl is-enabled ntpd 

[root@hadoop102 ~]# vi /etc/ntp.conf
```

取消注释 `restrict 192.168.217.0 mask 255.255.255.0 nomodify notrap`

指定网段上的所有机器可以从这台机器上查询和同步时间。

注释掉网络时间同步配置：

```shell
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
```

添加配置，采用本地时间作为时间服务器为集群中的其他节点提供时间同步

```shell
server 127.127.1.0
fudge 127.127.1.0 stratum 10
```

修改【/etc/sysconfig/ntpd】

```shell
vi /etc/sysconfig/ntpd

# 添加下面配置，硬件时钟和软件时间一起同步
SYNC_HWCLOCK=yes
```

配置启动

```shell
# 重启ntpd
systemctl stop ntpd
systemctl start ntpd

# 设置开机启动
systemctl enable ntpd
```

**其他节点配置**

在其他节点停止ntpd服务，并同步 【/etc/sysconfig/ntpd】 文件

```shell
[atguigu@hadoop103 ~]$ sudo systemctl stop ntpd
[atguigu@hadoop103 ~]$ sudo systemctl disable ntpd
[atguigu@hadoop104 ~]$ sudo systemctl stop ntpd
[atguigu@hadoop104 ~]$ sudo systemctl disable ntpd

# hadoop103和hadoop104 配置定时任务
[atguigu@hadoop103 ~]$ sudo crontab -e

# 维护内容如下
*/1 * * * * /usr/sbin/ntpdate hadoop102

```


