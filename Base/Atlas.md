<!-- TOC -->

- [Atlas](#atlas)
  - [安装配置](#安装配置)
    - [JDK](#jdk)
    - [maven](#maven)
    - [编译](#编译)
    - [HBase和Solr](#hbase和solr)
    - [启动](#启动)

<!-- /TOC -->

<a id="markdown-atlas" name="atlas"></a>
# Atlas

下载地址：

https://atlas.apache.org/#/Downloads

`apache-atlas-2.1.0-sources.tar.gz`

<a id="markdown-安装配置" name="安装配置"></a>
## 安装配置

<a id="markdown-jdk" name="jdk"></a>
### JDK

安装 OracleJDK，版本要求 jdk8+

<a id="markdown-maven" name="maven"></a>
### maven

http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz

【apache-maven-3.6.3-bin.tar.gz】解压并修改名称为【maven】，路径为：【/usr/local/maven】

```shell
/usr/local/src
rz

tar -zxvf apache-maven-3.6.3-bin.tar.gz
mv apache-maven-3.6.3 /usr/loacl/maven
```

配置环境变量：

```shell
vi /etc/profile
```

```
# maven环境变量配置
export M2_HOME=/usr/local/maven
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
```

```shell
souce /etc/profile
```

修改阿里云镜像

```shell
vi /usr/local/maven/conf/settings.xml
```

```xml

```

<a id="markdown-编译" name="编译"></a>
### 编译

```shell
cd /opt/apache-atlas-sources-2.1.0
mvn clean -DskipTests package -Pdist,embedded-hbase-solr
```

编译较慢，需要下载 HBase ，因为 HBase 下载不完全导致执行报错，如下所示：

```shell
[INFO] Apache Atlas Distribution 2.1.0 .................... FAILURE [  4.113 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  02:59 min
[INFO] Finished at: 2020-12-15T13:15:06+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (hbase) on project atlas-distro: An Ant BuildException has occured: Error while expanding /opt/apache-atlas-sources-2.1.0/distro/hbase/hbase-2.0.2.tar.gz
[ERROR] java.io.EOFException: Unexpected end of ZLIB input stream
[ERROR] around Ant part ...<untar src="/opt/apache-atlas-sources-2.1.0/distro/hbase/hbase-2.0.2.tar.gz" dest="/opt/apache-atlas-sources-2.1.0/distro/target/hbase.temp" compression="gzip"/>... @ 7:164 in /opt/apache-atlas-sources-2.1.0/distro/target/antrun/build-Download HBase.xml
```

<a id="markdown-hbase和solr" name="hbase和solr"></a>
### HBase和Solr

https://mirror.bit.edu.cn/apache/hbase/2.2.6/hbase-2.2.6-bin.tar.gz

https://mirror.bit.edu.cn/apache/lucene/solr/7.7.3/solr-7.7.3.tgz

https://mirror.bit.edu.cn/apache/lucene/solr/8.7.0/solr-8.7.0.tgz

https://npm.taobao.org/mirrors/node/v12.16.0/node-v12.16.0-linux-x64.tar.gz

修改`pom.xml`中版本配置：

```shell
cd /opt/apache-atlas-sources-2.1.0
```

```xml
<hbase.version>2.2.6</hbase.version>
<solr.version>8.7.0</solr.version>


<zookeeper.version>3.5.5</zookeeper.version>
```

可以手动下载 HBase 和 Solr 上传至指定目录


<a id="markdown-启动" name="启动"></a>
### 启动

```shell
cd /opt/apache-atlas-sources-2.1.0/distro/target/apache-atlas-2.1.0-bin/apache-atlas-2.1.0/bin

./atlas_start.py
```







---

参考引用：

[CentOS安装Maven](https://www.cnblogs.com/bincoding/p/6156236.html)

[apache atlas 2.0 详细安装手册](https://blog.csdn.net/coderblack/article/details/104283606)

[元数据与数据治理|Apache Atlas安装过程详解（初步版本）](https://blog.csdn.net/zzhuan_1/article/details/86170955)




