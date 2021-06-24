---
title: Spark30天教程笔记
typora-root-url: ../../source
date: 2021-06-23 22:24:06
tags: spark
categories: 大数据
sticky:
---

# 001_spark简介

## 什么是spark

官网介绍：

> Lightning-fast , unified, analytics engine
>
> 快，统一的，计算/分析引擎

- Speed 速度快：

  官网上唬人的广告：spark和Hadoop（mapreduce）相比快100倍。这个其实标注了特定场景，逻辑回归场景下，也就是多次迭代情况下表现比较好。

  Spark的快是由于**基于内存+DAG scheduler任务调度**：

  Spark相比MR而言，工作流有差异。Spark有DAG，在数据不需要真正落地之前，能缓存在内存中就都将数据缓存在内存中；MR的多个map和reduce任务都是独立的概念，在相互进行数据交互的过程中，都涉及到数据的持久化（落地磁盘）。即Spark的基于内存其实是数据交互不走磁盘。（tez就是MR+DAG）。

- Ease of use

  支持多种语言，支持SQL

- Generality 一站式开发

  sql，spark streaming，机器学习、图计算

- Runs Everywhere

  yarn是hadoop的插槽，是hadoop的资源管理框架。spark如果要跑在yarn上需要实现接口application master（AM，是非常驻进程。另外resource master，RM是常驻进程。）local是本地模式，standalone是spark自带的资源管理框架。

## Spark & MapReduce的区别

**MapReduce：**

数据倾斜？

Map任务结束一次落地，reduce任务一次拉取，中间还有一次网络IO

Combiner：在map阶段的reduce，减少**网络IO**的开销（只有求最大值、求和等可以做，求平均不能做）

**Spark:**

执行流程大量减少磁盘IO，任务通过DAG规划具备连续性

**总结：Spark为什么快——内存+DAG**

## spark 运行模式

- local：多用于本地测试，如在idea中写程序测试

- standalone：spark自带资源调度框架

  - 调度的对象包括**资源和任务**两方面

- yarn：Hadoop生态圈的一个资源调度框架

  如果要基于Yarn进行资源调度，必须实现ApplicationMaster接口，spark实现了这个接口，所以可以基于Yarn来进行资源调度。

- mesos：也是资源调度框架。

# 002_Spark核心RDD

## RDD

RDD 弹性分布式数据集

### 五大属性

- RDD是由一系列的partition组成的
- 函数是作用在每一个partition(split)上的
- RDD之间是由一系列的依赖关系
- 分区器是作用在（K，V）格式的RDD上 （optional）
- RDD提供一系列最佳的计算位置（optional）![image-20210624204727728](/images/Spark30%E5%A4%A9%E6%95%99%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20210624204727728-1624538851306.png)

### WordCount程序编写

- 导入源码方便经常查看

- 三段式代码：

  a. 环境构建SparkConf(设置相关配置信息)--SparkContext；

  b. spark处理逻辑；

  c. 关闭SparkContext















































