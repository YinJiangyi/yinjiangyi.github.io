---
title: spark视频教程30天笔记
typora-root-url: ../../source
date: 2021-06-24 14:20:03
tags: spark
catagories: 大数据
sticky:
---

### 002_spark核心RDD

### spark 运行模式

- local：多用于本地测试，如在idea中写程序测试

- standalone：spark自带资源调度框架

  - 调度的对象包括**资源和任务**两方面

- yarn：Hadoop生态圈的一个资源调度框架

  如果要基于Yarn进行资源调度，必须实现ApplicationMaster接口，spark实现了这个接口，所以可以基于Yarn来进行资源调度。

- mesos：也是资源调度框架。



### spark core

#### RDD 弹性分布式数据集

##### RDD五大属性：

- RDD是由一系列的partition组成的
- 函数是作用在每一个partition(split)上的
- RDD之间是由一系列的依赖关系
- 分区器是作用在（K，V）格式的RDD上 （optional）
- RDD提供一系列最佳的计算位置（optional）

#### RDD源码介绍

![image-20210624144855109](/images/spark%E8%A7%86%E9%A2%91%E6%95%99%E7%A8%8B30%E5%A4%A9%E7%AC%94%E8%AE%B0/image-20210624144855109.png)













































