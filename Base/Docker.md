<!-- TOC -->

- [docker](#docker)
  - [前提](#前提)
  - [前期准备](#前期准备)
  - [docker安装](#docker安装)
  - [docker 源配置](#docker-源配置)
  - [docker查看状态](#docker查看状态)
  - [常用命令](#常用命令)
  - [jdk](#jdk)
  - [gitlab 安装](#gitlab-安装)
  - [mysql安装](#mysql安装)

<!-- /TOC -->

<a id="markdown-docker" name="docker"></a>
# docker

<a id="markdown-前提" name="前提"></a>
## 前提

docker中容器和镜像是很基础重要的概念，相当于面向对象的类和对象关系。

面向对象：类创建对象

docker：镜像image创建容器container


<a id="markdown-前期准备" name="前期准备"></a>
## 前期准备

```shell
cd /etc/yum.repos.d

# 备份默认的yum配置
mv CentOS-Base.repo CentOS-Base.repo.bak

# 下载新的yum配置
wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# 重新加载yum源
yum clean all
yum makecache
```

<a id="markdown-docker安装" name="docker安装"></a>
## docker安装

检查内核，必须大于3.1

```shell
uname -r
```

yum 安装 docker

```shell
# 安装yum工具
yum install yum-utils -y

# 增加aliyun镜像，提速
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装 docker-ce 社区版本
yum -y install docker-ce
```

<a id="markdown-docker-源配置" name="docker-源配置"></a>
## docker 源配置

在文件【/etc/docker/daemon.json】中配置国内源，如果没有该文件则创建

```shell
vi /etc/docker/daemon.json
```

添加一下国内源配置：
```json
{
 "registry-mirrors" : [
   "https://mirror.ccs.tencentyun.com",
   "http://registry.docker-cn.com",
   "http://docker.mirrors.ustc.edu.cn",
   "http://hub-mirror.c.163.com"
 ]
}
```


```shell
# 重启docker加载配置
service docker restart

# 查看 Registry Mirrors 是否已添加
docker info
```

<a id="markdown-docker查看状态" name="docker查看状态"></a>
## docker查看状态

```shell
# 查看状态
systemctl status docker 

# 开机启动
systemctl enable docker

systemctl start docker
systemctl restart docker
systemctl stop docker
```

<a id="markdown-常用命令" name="常用命令"></a>
## 常用命令

```
# 搜索相关镜像
docker search xxx

# 查看所有已下载镜像
docker images

# 按镜像id进行删除
docker image rm imageId

# 从远程仓库中拉取镜像，如果没有指定TAG,默认为latest
docker pull [镜像名]:[TAG]

# 移除本地镜像
docker rmi [镜像id...]

# 新建并启动容器
docker run
-d: 后台运行容器，并返回容器ID；
-i: 以交互模式运行容器，通常与 -t 同时使用；
-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

-P: 随机端口映射，容器内部端口随机映射到主机的端口
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

--name="nginx-lb": 为容器指定一个名称；

-m :设置容器使用内存最大值；


# 查看所有容器
docker ps -a

# 通过容器id启动容器
docker start [容器名/容器id]

# 停止容器
docker stop [容器名]

# 强制停止容器
docker kill [容器名]

# 删除单个容器
docker rm [容器名/容器id]

# 容器日志查看
docker log -f -t --tail [行数] 容器id
-t：在日志中加入时间戳
-f：跟随最新的日志打印
--tail：显示最后多少条日志

# 容器内运行的进程
docker top 容器名/id

# 进入正在运行的容器并以命令行交互
docker exec -it [容器名/id] /bin/bash

# 将容器中文件拷贝到主机中
docker cp 容器id:容器内部路径 目的主机路径

```


<a id="markdown-jdk" name="jdk"></a>
## jdk

```shell
# 拉取jdk镜像 
docker pull primetoninc/jdk:1.8

# 后台方式运行
docker run -dit --name jdk8 --restart=always primetoninc/jdk:1.8 

# 正在运行的容器
docker ps

# 命令方式进去容器
docker exec -it jdk8 /bin/bash

```

<a id="markdown-gitlab-安装" name="gitlab-安装"></a>
## gitlab 安装

```shell
# 拉取镜像
docker pull gitlab/gitlab-ce

# 查看已下载镜像
docker images
```

```shell
# 创建config目录
mkdir -p /usr/local/gitlab/config

# 创建logs目录
mkdir -p /usr/local/gitlab/logs

# 创建data目录
mkdir -p /usr/local/gitlab/data
```

执行如下命令，启动 docker 镜像

```shell
docker run --detach \
    --hostname 192.168.217.100 \
    --publish 10000:80 \
    --name "my-gitlab" --restart always \
    --volume /usr/local/gitlab/config:/etc/gitlab \
    --volume /usr/local/gitlab/data:/var/opt/gitlab \
    --volume /usr/local/gitlab/logs:/var/log/gitlab \
    gitlab/gitlab-ce
```


```shell
# 查看 docker 当前正在运行的镜像
docker ps

# 停止 docker 容器
docker stop containerId

# 查看所有 docker，包括
docker ps -a

# 根据 容器Id 启动容器
docker start containerId
```

<a id="markdown-mysql安装" name="mysql安装"></a>
## mysql安装

```shell
# 拉取 mysql 镜像
docker pull mysql
```

启动容器

```
-i：以交互模式运行，通常配合-t
-t：为容器重新分配一个伪输入终端，通常配合-i
-d：后台运行容器
-p：端口映射，格式为主机端口:容器端口
-e：设置环境变量，这里设置的是root密码
--name：设置容器别名
```

```shell
docker run -itd -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name "im-mysql" mysql
```


