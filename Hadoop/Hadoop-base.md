<!-- TOC -->

- [hadoop](#hadoop)
  - [生态体系](#生态体系)
  - [集群安装](#集群安装)
    - [基础环境](#基础环境)
    - [hadoop安装](#hadoop安装)
  - [应用](#应用)
    - [运行模式](#运行模式)
    - [脚本下发](#脚本下发)

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

前期ip网络、host配置

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

设置软件库

```shell
# 针对rhel配置软件库：
yum install -y epel-release

# 关闭防火墙
systemctl stop firewalld
# 关闭开机启动
systemctl disable firewalld.service
```

卸载虚拟机自带的 jdk 

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
tar -zxvf jdk-8u202-linux-x64.tar.gz -C /opt

# 配置环境变量
vi /etc/profile.d/my_env.sh
```

【my_env.sh】编辑内容：

```shell
# JAVA_HOME
export JAVA_HOME=/opt/jdk1.8.0_202
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
tar -zxvf /opt/software/hadoop-3.1.3.tar.gz -C /opt/

# pwd
/opt/hadoop-3.1.3

# vi /etc/profile.d/my_env.sh
```

添加以下配置：

```shell
# HADOOP_HOME
export HADOOP_HOME=/opt/hadoop-3.1.3
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
This command was run using /opt/hadoop-3.1.3/share/hadoop/common/hadoop-common-3.1.3.jar

```

hadoop 重要目录：

1. bin目录：存放对Hadoop相关服务（hdfs，yarn，mapred）进行操作的脚本
2. etc目录：Hadoop的配置文件目录，存放Hadoop的配置文件
3. lib目录：存放Hadoop的本地库（对数据进行压缩解压缩功能）
4. sbin目录：存放启动或停止Hadoop相关服务的脚本
5. share目录：存放Hadoop的依赖jar包、文档、和官方案例

<a id="markdown-应用" name="应用"></a>
## 应用

<a id="markdown-运行模式" name="运行模式"></a>
### 运行模式

Hadoop运行模式包括：本地模式、伪分布式模式以及完全分布式模式。

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

免密登录设置

```shell
# 远程登录 hadoop103 机器，会生成 ~/.ssh 隐藏文件夹
ssh hadoop103

cd ~/.ssh

# 生成公钥和密钥
ssh-keygen -t rsa

# 拷贝公钥到免密登录的机器
ssh-copy-id hadoop103
ssh-copy-id hadoop104

```



