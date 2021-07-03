---
title: Spark30天教程笔记
typora-root-url: ..
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

local：多用于本地测试，如在idea中写程序测试
standalone：spark自带资源调度框架

> 调度的对象包括**资源和任务**两方面

yarn：Hadoop生态圈的一个资源调度框架

> 如果要基于Yarn进行资源调度，必须实现ApplicationMaster接口，spark实现了这个接口，所以可以基于Yarn来进行资源调度。

mesos：也是资源调度框架。

# 002_Spark核心

## RDD

RDD 弹性分布式数据集

### 五大属性

1) RDD是由一系列的partition组成的
2) 函数是作用在每一个partition(split)上的
3) RDD之间有一系列的依赖关系
4) 分区器是作用在（K，V）格式的RDD上 （optional）
5) RDD提供一系列最佳的计算位置（optional）

源码介绍原文：![image-20210624204727728](/images/Spark30%E5%A4%A9%E6%95%99%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20210624204727728-1624538851306.png)

### WordCount程序编写

- 导入源码方便经常查看
- 三段式代码：
a. 环境构建SparkConf(设置相关配置信息)--SparkContext；
  b. spark处理逻辑；
c. 关闭SparkContext

### 总结整理

spark读取文件是使用sparkContext对象调用textFile方法，实际上底层和MR读取hdfs文件的方式是一样的，读取之前先要进行partition切片，默认情况下partition的大小和block块大小相同。另外都是行读取器，是以行为单位进行读取的。

#### RDD的分布式体现在哪里？
RDD是由一系列partition组成，partition是分布在不同的计算节点上的。

#### RDD的弹性体现在哪里？
RDD由一系列partition组成，partition的大小和数量都是可以改变的。默认情况下partition的个数和block块的个数相同。

- **block和partition的关联和区别**

  block是hdfs分布式存储的最小单元，类似存放文件的盒子，但是一个盒子的内容只可能来自同一个文件。假设block设置为128M，你的文件是250M，那么这份文件占3个block（128+128+2）。这样的设计虽然会有一部分磁盘空间的浪费，但是整齐的block大小，便于快速找到、读取对应的内容。（p.s. 考虑到hdfs冗余设计，默认三份拷贝，实际上3*3=9个block的物理空间。）

  partition是spark的弹性分布式数据集RDD的最小单元，RDD是由多个partition组成。partition 是指的spark在计算过程中，生成的数据在计算空间内最小单元，同一份数据（RDD）的partition 大小不一，数量不定，是根据application里的算子和最初读入的数据分块数量决定的，这也是为什么叫“弹性分布式”数据集的原因之一。

  > **总结：**
  > ***block位于存储空间、partition 位于计算空间，***
  > ***block的大小是固定的、partition 大小是不固定的，***
  > ***block是有冗余的、不会轻易丢失，partition（RDD）没有冗余设计、丢失之后重新计算得到***

#### RDD的容错体现在哪里？
（容错也是弹性体现的一个方面）
RDD之间存在依赖关系，子RDD可以找对应的父RDD然后通过一系列计算再得出相应的结果。如果计算过程中某一个rdd挂掉了，不需要从头开始重新计算。

> `虽然从代码上看，RDD像是存储数据的结构，但实际上RDD是一个抽象的概念，存储的是计算逻辑，并不存储物理数据。`

## Spark任务执行原理

### Standalone

- driver & worker & master

  - driver和worker都是jvm进程，可以通过jps命令查看
  - master和worker的架构是**主从架构**，“一主多从”架构会存在**单点故障**问题

  - driver和集群之间有频繁的通信
  - driver负责任务的分发和结果的回收，即任务的调度。如果task的计算结果非常大，就不要回收了，会造成out of memory

  > 什么叫节点的**资源**：文件资源（data），内存资源（RAM），计算资源（CPU）

## Spark代码流程

1. 创建SparkConf对象

   本质上就是spark任务的相关配置，可以设置app name，运行模式和资源需求等

2. 创建SparkContext对象

3. 基于SparkContext即上下文环境对象，创建一个RDD，对RDD进行处理。

4. 应用程序中要有Action算子（触发算子）来触发Transformation算子执行

5. 关闭Spark上下文对象

## 常见算子

（具体使用见代码 ）

### 转换算子/Transformations

懒加载执行

rdd->rdd

### 触发算子/Actions

rdd->其他

launch a job to return a value to the user program

所有的action类的算子作为计算的触发条件。

### 持久化算子/控制算子

将rdd持久化，持久化的单位是partition

控制算子有三种：cache，persist，checkpoint
其中cache是persist中的一个，类似first和take中的一个

（代码演示）

#### cache算子

![image-20210630221616093](/images/Spark30%E5%A4%A9%E6%95%99%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20210630221616093.png)

如上图，原来的代码逻辑wordcount是rdd3->rdd4，后来加了rdd5，这时rdd5的操作触发后不是直接从rdd3过来的，而是从txt文件读取开始，所以这个时候需要对rdd3进行持久化（cache)

#### persist算子

可以指定持久化的级别。最常用memory_only和memory_and_disk

#### cache和persist联系和区别

- cache和persist的注意事项

  1. 都是懒执行，必须有一个触发算子；
  2. 但不能紧跟action算子，只能下一行，编译不会通过
  3. 返回值可以赋值给一个变量，在其他job中直接使用这个变量就是使用持久化之后的数据。持久化的单位是partition。

- cache和persist的区别

  cache调用persist，默认storageLevel是memory_only，而persist可以指定level参数

- storageLevel：
  	最常用的两个：

  ​	memory_only，性能最高
  ​	memory_and_disk（解决oom）

  ​	性能依次往下选择：memory，ser，disk

  > ser序列化参数：放内存或放磁盘时字节数会小一些，可更大程度避免oom或者频繁的gc）

  > 源码storageLevel中，**带_2后缀（存副本）的选项一般不选。**
  >
  > 这涉及持久化算子的两个功能：1）性能调优 2）持久化容错
  >
  > ​	其中cache和persist是为了性能调优，所以主要选择memory

#### checkpoint算子

主要作用是容错，将rdd直接持久化到磁盘。

源码查看：

```java
/**
 * Mark this RDD for checkpointing. It will be saved to a file inside the checkpoint
 * directory set with SparkContext.setCheckpointDir() and all references to its parent
 * RDDs will be removed. This function must be called before any job has been
 * executed on this RDD. It is strongly recommended that this RDD is persisted in
 * memory, otherwise saving it on a file will require recomputation.
 */
def checkpoint(): Unit = {
  rdd.checkpoint()
}
```

这里表示，需要在context中进行checkpoint路径设置setCheckpointDir。

执行原理：

1. 当rdd的job执行完毕之后，会从finalrdd从后往前回溯。
2. 当回溯到某一个rdd调用了checkpiont方法，会对当前的rdd做一个标记
3. spark框架会**自动启动一个新的job**，重新计算这个rdd的数据，将数据持久化到hdfs上。

使用checkpoint常用的优化手段：**对rdd执行checkpoint之前，最好对这个rdd先执行cache**，这样新启动job只需要将内存中的数据拷贝到hdfs中就可以了，省去了重新计算的步骤。

> 有诸多不变，逻辑和数据都保留下来了，如果代码改变了会造成问题。

#### pipeline

![image-20210630230624396](/images/Spark30%E5%A4%A9%E6%95%99%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20210630230624396.png)

# 复习day1

![spark](/images/Spark30%E5%A4%A9%E6%95%99%E7%A8%8B%E7%AC%94%E8%AE%B0/spark.png)

# 003 Standalone模式

## standalone模式集群搭建

Window 10 

- Vmware安装
- ubuntu虚拟机安装（三个节点）
- 设置成桥接模式

> 三个node IP分别为192.168.124.6-8，主机ip地址为192.168.124.5，具体设置内容查看“VMware三种联网方式及原理”





















