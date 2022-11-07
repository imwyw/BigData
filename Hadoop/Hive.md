<!-- TOC -->

- [Hive](#hive)
  - [环境准备](#环境准备)
    - [基础包准备](#基础包准备)
    - [hive安装](#hive安装)
    - [元数据库替换](#元数据库替换)
    - [启动hive](#启动hive)
      - [hive配置](#hive配置)
      - [hive初始化](#hive初始化)
      - [jdbc访问hive](#jdbc访问hive)

<!-- /TOC -->

<a id="markdown-hive" name="hive"></a>
# Hive

<a id="markdown-环境准备" name="环境准备"></a>
## 环境准备

<a id="markdown-基础包准备" name="基础包准备"></a>
### 基础包准备

hive下载地址：

http://archive.apache.org/dist/hive/

元数据metastore替换数据库mysql下载：

https://downloads.mysql.com/archives/community/

https://downloads.mysql.com/archives/c-j/

<a id="markdown-hive安装" name="hive安装"></a>
### hive安装

把 apache-hive-3.1.2-bin.tar.gz 上传到 linux 的/opt/software 目录下

```shell
# 解压 到/opt/module/目录下面
tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/

# 重命名为 hive
mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive

# 修改添加环境变量
sudo vi /etc/profile.d/my_env.sh

```

添加内容：

```shell
# HIVE_HOME
export HIVE_HOME=/opt/module/hive
export PATH=$PATH:$HIVE_HOME/bin
```


```shell
# 生效环境变量
source /etc/profile

# 解决日志 Jar 包冲突
mv $HIVE_HOME/lib/log4j-slf4j-impl-2.17.1.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.17.1.bak
```

<a id="markdown-元数据库替换" name="元数据库替换"></a>
### 元数据库替换

```shell
# 检查当前系统是否安装过 MySQL
rpm -qa|grep mariadb
# mariadb-libs-5.5.65-1.el7.x86_64

# 卸载已存在的 mariadb-libs
sudo rpm -e --nodeps mariadb-libs

# 创建mysql文件夹
mkdir /opt/software/mysql/

# 解压 mysql bundle 包
tar -xf /opt/software/mysql-5.7.39-1.el7.x86_64.rpm-bundle.tar -C /opt/software/mysql/

# 按顺序安装mysql包
cd /opt/software/mysql
sudo rpm -ivh mysql-community-common-5.7.39-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.39-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.39-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-client-5.7.39-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-server-5.7.39-1.el7.x86_64.rpm


# 删除/etc/my.cnf 文件中 datadir 指向的目录下的原有内容
[mysqld]
datadir=/var/lib/mysql

# 初始化数据库
sudo mysqld --initialize --user=mysql

# 查看临时生成的 root 用户的密码
sudo cat /var/log/mysqld.log

# 启动 mysql 服务
sudo systemctl start mysqld

# 使用默认密码登录数据库
mysql -uroot -p
Enter password: 输入临时生成的密码...

# 修改 mysql root 用户的默认密码，否则后续操作会报错
mysql> set password = password("新密码");

# 修改 mysql 库下的 user 表中的 root 用户允许任意 ip 连接
mysql> update mysql.user set host='%' where user='root';
mysql> flush privileges;

```

通过datagrip或者navict进行客户端连接测试验证。

<a id="markdown-启动hive" name="启动hive"></a>
### 启动hive

<a id="markdown-hive配置" name="hive配置"></a>
#### hive配置

```shell
# 拷贝驱动到hive的lib目录
cp /opt/software/mysql-connector-java-5.1.49.jar $HIVE_HOME/lib
```

配置 Metastore 到 MySQL

```shell
vi $HIVE_HOME/conf/hive-site.xml
```

【hive-site.xml】内容如下：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<!-- jdbc 连接的 URL -->
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://hadoop102:3306/metastore?useSSL=false</value>
	</property>
	<!-- jdbc 连接的 Driver-->
	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.jdbc.Driver</value>
	</property>
	<!-- jdbc 连接的 username-->
	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>root</value>
	</property>
	<!-- jdbc 连接的 password -->
	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>123456</value>
	</property>
	<!-- Hive 元数据存储版本的验证 -->
	<property>
		<name>hive.metastore.schema.verification</name>
		<value>false</value>
	</property>
	<!--元数据存储授权-->
	<property>
		<name>hive.metastore.event.db.notification.api.auth</name>
		<value>false</value>
	</property>
	<!-- Hive 默认在 HDFS 的工作目录 -->
	<property>
		<name>hive.metastore.warehouse.dir</name>
		<value>/user/hive/warehouse</value>
	</property>
</configuration>
```

<a id="markdown-hive初始化" name="hive初始化"></a>
#### hive初始化

```shell
mysql -uroot -p123456

# 新建 Hive 元数据库
create database metastore;
quit;

# 初始化 Hive 元数据库
schematool -initSchema -dbType mysql -verbose

```

初始化Hive仓库报错

Exception in thread “main” java.lang.NoSuchMethodError: 
com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V

com.google.common.base.Preconditions.checkArgument 这是因为hive内依赖的guava.jar和hadoop内的版本不一致造成的。

- hive中guava.jar位置/hive/lib/
- hadoop中guava.jar位置/hadoop/share/hadoop/common/lib/

删除版本低的，换成其中一个的高版本的！！！

```shell
# 统一使用 hadoop 的高版本
cp $HADOOP_HOME/share/hadoop/common/lib/guava-27.0-jre.jar $HIVE_HOME/lib/
# 删除 hive 的低版本
rm $HIVE_HOME/lib/guava-19.0.jar
# 重新进行初始化
schematool -initSchema -dbType mysql -verbose
```

```shell
cd $HIVE_HOME
# 启动hive
bin/hive

# hive操作
hive> show databases;
hive> show tables;
hive> create table test (id int);
hive> insert into test values(1);
hive> select * from test;
```

<a id="markdown-jdbc访问hive" name="jdbc访问hive"></a>
#### jdbc访问hive

```shell
vi /opt/module/hive/conf/hive-site.xml
```

在 hive-site.xml 文件中添加如下配置信息

```xml
  <!-- 指定存储元数据要连接的地址 -->
  <property>
    <name>hive.metastore.uris</name>
    <value>thrift://hadoop102:9083</value>
  </property>
  
  <!-- 指定 hiveserver2 连接的 host -->
  <property>
    <name>hive.server2.thrift.bind.host</name>
    <value>hadoop102</value>
  </property>
  <!-- 指定 hiveserver2 连接的端口号 -->
  <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
  </property>
```

服务启动脚本【hiveservices.sh】

```shell
vi $HIVE_HOME/bin/hiveservices.sh
```

```shell
#!/bin/bash
HIVE_LOG_DIR=$HIVE_HOME/logs
if [ ! -d $HIVE_LOG_DIR ]
then
mkdir -p $HIVE_LOG_DIR
fi
#检查进程是否运行正常，参数 1 为进程名，参数 2 为进程端口
function check_process()
{
 pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
 ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
 echo $pid
 [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
}
function hive_start()
{
 metapid=$(check_process HiveMetastore 9083)
 cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 
&"
 [ -z "$metapid" ] && eval $cmd || echo "Metastroe 服务已启动"
 server2pid=$(check_process HiveServer2 10000)
 cmd="nohup hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
 [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2 服务已启动"
}
function hive_stop()
{
metapid=$(check_process HiveMetastore 9083)
 [ "$metapid" ] && kill $metapid || echo "Metastore 服务未启动"
 server2pid=$(check_process HiveServer2 10000)
 [ "$server2pid" ] && kill $server2pid || echo "HiveServer2 服务未启动"
}
case $1 in
"start")
 hive_start
 ;;
"stop")
 hive_stop
 ;;
"restart")
 hive_stop
 sleep 2
 hive_start
 ;;
"status")
 check_process HiveMetastore 9083 >/dev/null && echo "Metastore 服务运行正常" || echo "Metastore 服务运行异常"
 check_process HiveServer2 10000 >/dev/null && echo "HiveServer2 服务运行正常" || echo "HiveServer2 服务运行异常"
 ;;
*)
 echo Invalid Args!
 echo 'Usage: '$(basename $0)' start|stop|restart|status'
 ;;
esac
```

```shell
chmod +x $HIVE_HOME/bin/hiveservices.sh

# 启动 hive 后台服务
hiveservices.sh start
```


