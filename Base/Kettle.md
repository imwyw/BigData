<!-- TOC -->

- [Kettle](#kettle)
  - [环境](#环境)
  - [jenkins](#jenkins)
    - [安装下载](#安装下载)
    - [启动和停止](#启动和停止)

<!-- /TOC -->


<a id="markdown-kettle" name="kettle"></a>
# Kettle

Kettle 是一个组件化的集成系统，包括如下几个主要部分：

1. Spoon：图形化界面工具(GUI方式)， Spoon 允许你通过图形界面来设计 Job 和 Transformation ，可以保存为文件或者保存在数据库中。
2. Pan：Transformation执行器(命令行方式)，Pan用于在终端执行Transformation，没有图形界面。
3. Kitchen：Job执行器(命令行方式)，Kitchen用于在终端执行Job，没有图形界面。
4. Carte：嵌入式Web服务，用于远程执行Job或Transformation，Kettle通过Carte建立集群。
5. Encr：Kettle用于字符串加密的命令行工具，如：对在Job或Transformation中定义的数据库连接参数进行加密。



<a id="markdown-环境" name="环境"></a>
## 环境

依赖于 jdk，配置参见：

https://github.com/imwyw/Linux/blob/master/Basic/Dev.md

<a id="markdown-jenkins" name="jenkins"></a>
## jenkins

<a id="markdown-安装下载" name="安装下载"></a>
### 安装下载

<a id="markdown-启动和停止" name="启动和停止"></a>
### 启动和停止





--- 

[kettle学习笔记（一）——入门与安装](https://www.cnblogs.com/jiangbei/p/8717853.html)

[kettle学习笔记（二）——kettle基本使用](https://www.cnblogs.com/jiangbei/p/8985894.html)

[kettle学习笔记（三）——kettle资源库、运行方式与日志](https://www.cnblogs.com/jiangbei/p/8987403.html)

[kettle学习笔记（四）——kettle输入步骤](https://www.cnblogs.com/jiangbei/p/8989347.html)

[kettle学习笔记（五）——kettle输出步骤](https://www.cnblogs.com/jiangbei/p/8994145.html)

[kettle学习笔记（六）——kettle转换步骤](https://www.cnblogs.com/jiangbei/p/8995342.html)

[kettle学习笔记（七）——kettle流程步骤与应用步骤](https://www.cnblogs.com/jiangbei/p/8996386.html)

[kettle学习笔记（八）——kettle查询步骤与连接步骤](https://www.cnblogs.com/jiangbei/p/8997402.html)

[kettle学习笔记（九）——子转换、集群与变量](https://www.cnblogs.com/jiangbei/p/8999634.html)

[kettle学习笔记（十）——数据检验、统计、分区与JS脚本](https://www.cnblogs.com/jiangbei/p/9002062.html)


