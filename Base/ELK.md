# ELK

## docker-elk 

```shell
docker pull sebp/elk:660

# 最大内存 512Mb
echo "vm.max_map_count=524288" > /etc/sysctl.conf

sysctl -p

# 创建容器 并设置映射端口
docker run -dit --name elk \
    -p 5601:5601 \
    -p 9200:9200 \
    -p 5044:5044 \
    -v /opt/elk-data:/var/lib/elasticsearch \
    -v /etc/localtime:/etc/localtime \
    sebp/elk:660

# 动态修改为 自动重启
docker update --restart=always elk

```

- 9200：elasticsearch端口
- 5601：kibana端口
- 5044：logstash beat端口

## kibana



## logstash

Logstash有两种配置文件：
- 管道配置文件，它定义Logstash处理管道；
- 设置文件，它指定控制Logstash启动和执行的选项。


### 管道配置文件

```shell
# logstash 配置
cd /etc/logstash/conf.d

root@f9b85ba14ea2:/etc/logstash/conf.d# ll
-rw-r--r-- 1 root     root     268 Jul  4 17:09 02-beats-input.conf
-rw-r--r-- 1 root     root     177 Jul  4 16:49 02-beats-input.conf.back
-rw-r--r-- 1 root     root     456 Feb  6  2019 10-syslog.conf
-rw-r--r-- 1 root     root     113 Feb  6  2019 11-nginx.conf
-rw-r--r-- 1 root     root     184 Feb  6  2019 30-output.conf
-rw-r--r-- 1 root     root     184 Jul  4 17:12 30-output.conf.back
```

Logstash尝试在/etc/logstash/conf.d目录中只加载扩展名为.conf的文件并忽略所有其他文件，并进行合并

### 设置文件

```
cd /opt/logstash/config

root@f9b85ba14ea2:/opt/logstash/config# ll
-rw-r--r-- 1 logstash logstash 1846 Jan 24  2019 jvm.options
-rw-r--r-- 1 logstash logstash 4568 Jan 24  2019 log4j2.properties
-rw-r--r-- 1 root     root      436 Jul  4 15:37 logstash-sample.conf
-rw-r--r-- 1 root     root      342 Jul  4 11:15 logstash-sample.conf.back
-rw-r--r-- 1 root     root     8162 Jul  4 11:13 logstash.yml
-rw-r--r-- 1 root     root     8162 Jul  4 10:54 logstash.yml.back
-rw-r--r-- 1 logstash logstash  285 Feb  6  2019 pipelines.yml
-rw-r--r-- 1 logstash logstash 1696 Jan 24  2019 startup.options
```

- logstash.yml

包含Logstash配置标志，你可以在这个文件中设置标志，而不是在命令行中传递标志，在命令行中设置的任何标志都覆盖logstash.yml文件中的相应设置，更多信息见logstash.yml。

- pipelines.yml

包含在一个Logstash实例中运行多个管道的框架和说明，更多信息请参见多个管道。

- jvm.options

包含JVM配置标志，使用此文件为总堆空间设置初始值和最大值，你还可以使用此文件为Logstash设置locale，在单独的行上指定每个标志，此文件中的所有其他设置都被视为专业设置。

- log4j2.properties

包含log4j 2库的默认设置，有关更多信息，请参见Log4j 2配置。

- startup.options

包含在/usr/share/logstash/bin中使用的system-install脚本选项，以便为你的系统构建适当的启动脚本。当你安装Logstash包时，system-install脚本在安装过程的末尾执行，并使用在startup.options中指定的设置来设置如用户、组、服务名和服务描述的选项。默认情况下，Logstash服务被安装在用户logstash下，startup.options文件使你更容易安装Logstash服务的多个实例，你可以复制文件并更改特定设置的值。注意，startup.options文件不是在启动时读取的，如果你想要更改Logstash启动脚本（例如，要更改Logstash用户或从不同的配置路径读取），你必须重新运行system-install脚本（作为root）以传递新的设置。

## springboot日志解析

### springboot项目配置

pom添加依赖项：

```xml
<!-- elk 日志收集开始 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    <version>2.5.2</version>
</dependency>
<!--集成logstash-->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>5.3</version>
</dependency>
<!-- elk 日志收集结束 -->
```

修改yml配置：

```yml

spring:
  data:
    elasticsearch:
      repositories:
        enabled: true #打开es仓库
  elasticsearch:
    rest:
      uris: 192.168.217.150:9200 # es链接地址

logstash:
  host: 192.168.217.150 # logstash 部署的服务器
  env: local,test,prod

```

logback-spring.xml配置

```xml
  <!--应用名称-->
  <springProperty scope="context" name="APP_NAME" source="spring.application.name"
    defaultValue="${project.artifactId}"/>
  <!--LogStash访问host-->
  <springProperty name="LOG_STASH_HOST" scope="context" source="logstash.host"
    defaultValue="192.168.217.150"/>

  <!--INFO日志输出到文件-->
  <appender name="LOG_STASH_INFO" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>${LOG_STASH_HOST}:5044</destination>
    <encoder chartset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder">
      <customFields>{"appname":"${APP_NAME}"}</customFields>
      <!--设置为默认的文件日志格式-->
      <providers>
        <pattern>
          <!--设置为默认的文件日志格式-->
          <pattern>%date [%thread] [%X{traceId}] - [ %-5level ] [%logger{50}] %file:%line - %msg%n
          </pattern>
        </pattern>
      </providers>
    </encoder>
  </appender>

```

### logstash管道配置

修改【/etc/logstash/conf.d/02-beats-input.conf】配置：

```shell
input {
  tcp {
    port => 5044 # logstash监听端口
    codec => json_lines
    #ssl => true
    #ssl_certificate => "/etc/pki/tls/certs/logstash-beats.crt"
    #ssl_key => "/etc/pki/tls/private/logstash-beats.key"
  }
}
```

修改【/etc/logstash/conf.d/30-output.conf】配置：

```shell
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    manage_template => false
    index => "%{[appname]}-%{+YYYY.MM.dd}"
    #document_type => "%{[@metadata][type]}"
  }
}
```

重启docker容器，logstash conf生效。
