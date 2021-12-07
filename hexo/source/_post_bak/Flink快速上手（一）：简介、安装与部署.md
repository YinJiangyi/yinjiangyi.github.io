---
title: Flink快速上手（一）：简介、安装与部署
date: 2021-01-28 15:06:06
tags: Flink
categories: 大数据
typora-root-url: ../../source
---

Apache Flink is a framework and distributed **processing engine** for **statefu**l computations over ***unbounded and bounded*** data **streams**.  <!--more-->

### 为什么选择Flink？

- 流式的、无限的数据更接近真实数据的产生方式
- 目标：低时延、高吞吐、准确和良好的容错

### 传统数据处理架构

- 事务处理OLTP（Online Transactional Processing）
  - 问题：多应用程序用到相同数据表或基础设施，更改和扩展难度大
  - 微服务架构
- 分析处理OLAP（Online Analytical Processing）
  - 汇总数据分析价值高，多个事务数据库会先进行汇总复制到数据仓库
  - ETL
  - 数据仓上执行两类查询，报表和ad-hoc查询

### 有状态流式处理演进

- 原理：有状态，checkpoint持久化 -> 低延时，结果不准确
- lambda框架：批+流；一个需求两个框架，融合性差
- flink：低延时（storm）、高吞吐（spark streaming微批）、压力下保持正确、时间正确、操作简单/表现力好

### Flink特点

- 事件驱动
- 基于流的世界观
- 分层api

### Flink & spark streaming

- 数据模型：spark-RDD

- 运行架构：spark-DAG(stage)



### Flink 快速上手

- IDEA中运行flink程序
- flink 部署
- web interface使用 & 终端命令行操作



---

### 实操

在aliyun服务器上安装部署flink及简单测试。

- 新建 `/root/flink`目录，`wget http://mirror.bit.edu.cn/apache/flink/flink-1.12.1/flink-1.12.1-bin-scala_2.11.tgz` 下载安装包。

- 解压 `tar -zxf flink-1.5.0-bin-hadoop26-scala_2.11.tgz`

- 启动flink集群报错 `./bin/stop-cluster.sh`

  `Please specify JAVA_HOME. Either in Flink config ./conf/flink-conf.yaml or as system-wide JAVA_HOME. `

  没有安装java环境，[安装java环境](https://www.jianshu.com/p/10949f44ce9c)

- 启动flink集群

  ```
  [root@izbp1a6f5b6v2v8osiww1vz flink-1.12.1]# ./bin/start-cluster.sh
  Starting cluster.
  Starting standalonesession daemon on host izbp1a6f5b6v2v8osiww1vz.
  Starting taskexecutor daemon on host izbp1a6f5b6v2v8osiww1vz.
  ```

  访问Web interface: `47.96.95.197:8081` 访问失败

  - 关闭防火墙 `systemctl stop firewalld.service` ，无效

  - 进入阿里云控制台后台，设置 安全->防火墙，添加规则开放所有tcp端口后，访问成功。

    <img src="/images/Flink-安装与部署/image-20210126184915758.png" alt="image-20210126184915758" style="zoom: 33%;" />

  