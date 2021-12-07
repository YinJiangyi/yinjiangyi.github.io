---
title: HBase上手记录
typora-root-url: ../../source
date: 2021-07-20 10:16:47
tags: HBase
categories: 数据库
sticky:
---

https://zhuanlan.zhihu.com/p/145551967

# 一、HBase是什么

> [Apache](https://link.zhihu.com/?target=https%3A//www.apache.org/) HBase™ is the [Hadoop](https://link.zhihu.com/?target=https%3A//hadoop.apache.org/) database, a distributed, scalable, big data store.
> HBase is a type of "NoSQL" database.

Apache HBase 是 Hadoop 数据库，一个分布式、可伸缩的**大数据存储**。

HBase是依赖Hadoop的。为什么HBase能存储海量的数据？**因为HBase是在HDFS的基础之上构建的，HDFS是分布式文件系统**。

# 二、为什么用HBase

和其他组件相比：

- Mysql？单机，存储数据量取决于硬盘大小，扩展性不好
- Kafka？主要用来处理消息（解耦、异步、削峰）。数据到kafka会被持久化到硬盘里，并且是分布式的，可以存储大量的数据，但是Kafka的数据一般不单独取出来。持久化了的数据，最常见的用法是重新设置offset，做回溯操作。
- Redis？内存速度快，价格太贵
- ES？Elasticsearch是分布式搜索引擎，但主要用于检索。如果没有经常见检索的需求，不必放到ES里，因为数据写入ES需要分词，浪费资源。
- HDFS？无疑可以存储海量数据，但是不支持随机修改，查询效率低

HBase的数据其实也是存储在HDFS上的。HDFS是文件系统，而HBase是数据库，其实也没啥可比性。「**你可以把HBase当做是MySQL，把HDFS当做是硬盘。HBase只是一个NoSQL数据库，把数据存在HDFS上**」。HBase在HDFS之上提供了**高并发的随机写和支持实时查询**，这是HDFS不具备的。HBase还有一个特点就是：**存储数据的”结构“可以地非常灵活**。

# 三、入门HBase

## 1. 列式存储

**什么是列式存储？**

MySQL存储的结构是典型的关系型数据库，数据是一行一行的形式。

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-58697c09af0a80f04af4b2231e6a0813_720w.jpg" alt="v2-58697c09af0a80f04af4b2231e6a0813_720w" style="zoom:25%;" />

换成列式存储

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-8b4c896c9decec36b89e6b2fee968c6a_720w.jpg" alt="v2-8b4c896c9decec36b89e6b2fee968c6a_720w" style="zoom:25%;" />

也就是把每列抽出来，关联上行ID。**这样做的好处是，省空间**。 以前我们一行记录多个属性(列)，有部分的列是空缺的，但是我们还是需要空间去存储。现在把这些列全部拆开，有什么我们就存什么，这样空间就能被我们充分利用。

这种形式更像**Key-value**。

## 2. HBase的数据模型

在HBase里边，定位一行数据会有一个唯一的值，这个叫做行键(RowKey)。而在HBase的列不是我们在关系型数据库所想象中的列。

HBase的列（Column）都得归属到列族（Column Family）中。在HBase中用列修饰符（Column Qualifier）来标识每个列。

**在HBase里边，先有列族，后有列**。

什么是列族？可以简单理解为：列的属性类别

什么是列修饰符？先有列族后有列，在列族下用列修饰符来标识一列。

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-1f69aa3a03c9666d5b613e5a0a5e70c4_720w.jpg" alt="v2-1f69aa3a03c9666d5b613e5a0a5e70c4_720w" style="zoom: 50%;" />

HBase表的每一行中，列的组成都是**灵活**的，**行与行之间的列不需要相同**。如图下：

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-fac481c6ec3efacce0ec3f3c7d127db0_720w.jpg" alt="img" style="zoom:33%;" />

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-de19abe0e27b1660e3474bd11aaf8d61_720w.jpg" alt="img" style="zoom: 50%;" />

换句话说：**一个列族下可以任意添加列，不受任何限制**

**数据写到HBase的时候都会被记录一个时间戳，这个时间戳被我们当做一个版本。比如说，我们修改或者删除某一条的时候，本质上是往里边新增一条数据，记录的版本加一了而已。**

比如现在我们有一条记录：

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-433053cc5a4dfe5a9d26f9231cdaca5e_720w.jpg" alt="img" style="zoom: 50%;" />

现在要把这条记录的值改为40，实际上就是多添加一条记录，在读的时候按照时间戳**读最新**的记录。在外界「看起来」就是把这条记录改了。

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-a08e56a695e366a580c12514ffe09a21_720w.jpg" alt="img" style="zoom:50%;" />

## 3. HBase的Key-Value结构

HBase的Key-Value结构图：

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-5c2c05489dc0c4af823d26641dc0e477_720w.jpg" alt="v2-5c2c05489dc0c4af823d26641dc0e477_720w" style="zoom: 50%;" />

> Key = Row Key + Column Family + Column Qualifier + Time stamp + Key Type
>
> Value是实际的字段值

## 4. HBase架构

架构图：

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-f9029a2beaf2b07d9ae949013ddca351_720w.jpg" alt="v2-f9029a2beaf2b07d9ae949013ddca351_720w" style="zoom: 67%;" />

1、**Client**客户端，它提供了访问HBase的接口，并且维护了对应的cache来加速HBase的访问。

2、**Zookeeper**存储HBase的元数据（meta表），无论是读还是写数据，都是去Zookeeper里边拿到meta元数据**告诉给客户端去哪台机器读写数据**

3、**HRegionServer**它是处理客户端的读写请求，负责与HDFS底层交互，是真正干活的节点。

4、HMaster处理 HRegion 的分配或转移。如果我们HRegion的数据量太大的话，HMaster会对拆分后的Region**重新分配RegionServer**。HMaster会处理元数据的变更和监控RegionServer的状态。

总结大致的流程就是：**client请求到Zookeeper，然后Zookeeper返回HRegionServer地址给client，client得到Zookeeper返回的地址去请求HRegionServer，HRegionServer读写数据后返回给client。**

HRegionServer内部结构图：

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/v2-a2bb5cd2216cb2b98d181d5eac1ca87f_720w.jpg" alt="v2-a2bb5cd2216cb2b98d181d5eac1ca87f_720w" style="zoom:80%;" />

- 一张hbase表按照RowKey进行横向切割，每个HRegionServer中包含多个`HRegion`，每个HRegion存储横向切割后的一部分数据。
- 每个HRegion中包含多个`store`，存储不同的列族。列是在列族之下的，列可以随意添加。
- HBase在写数据的时候，会先写到`Mem Store`，当MemStore超过一定阈值，就会将内存中的数据刷写到硬盘上，形成StoreFile，而`StoreFile`底层是以HFile的格式保存，HFile是HBase中KeyValue数据的存储格式。Mem Store我们可以理解为内存 buffer，HFile是HBase实际存储的数据格式，而StoreFile只是HBase里的一个名字。
- 为了防止机器宕机，内存的数据没刷到磁盘中就挂了。我们在写Mem store的时候还会写一份`HLog`。

# 四、HBase实际操作

实验环境中已经基于docker安装了Hbase环境。`docker exec` 命令直接进入容器。`jps` 命令查看运行进程：

```shell
[root@bigdata-12 /]# jps
34851 Jps
37 HMaster
123 HRegionServer
269 RESTServer
```

## 1. HBase shell操作

`hbase shell` 命令进入命令行，查看表格:

```shell
hbase(main):001:0> list
TABLE
aaa
bucket-0915
entity_cert_serial
entity_client_ip
entity_file
entity_fqdn
entity_mail
entity_qq_account
entity_server_ip
entity_user_id
……
```

创建表，表名hbase_1102，HBase表是由Key-Value组成的，此表中Key为NAME 

<img src="/images/HBase%E4%B8%8A%E6%89%8B%E8%AE%B0%E5%BD%95/726151-20170227162331001-824551785.png" alt="img" style="zoom:75%;" />

此表有两个列族，CF1和CF2，其中CF1和CF2下分别有两个列name和gender，Chinese和Math 
创建表hbase_1102有两个列族CF1和CF2

```
hbase(main):041:0> create 'hbase_1102',  {NAME=>'cf1'}, {NAME=>'cf2'}
```

向表中添加数据，在想HBase的表中添加数据的时候，只能一列一列的添加，不能同时添加多列。

```
hbase(main):042:0> put'hbase_1102', '001','cf1:name','Tom'
hbase(main):043:0> put'hbase_1102', '001','cf1:gender','man'
hbase(main):044:0> put'hbase_1102', '001','cf2:chinese','90'
hbase(main):045:0> put'hbase_1102', '001','cf2:math','91'
```

这样表结构就起来了，其实比较自由，列族里边可以自由添加子列很方便。如果列族下没有子列，加不加冒号都是可以的。

如果在添加数据的时候，需要手动的设置时间戳，则在put命令的最后加上相应的时间戳，时间戳是long类型的，所以不需要加引号

```
hbase(main):045:0> put'hbase_1102', '001','cf2:math','91'，1478053832459
```

查看表中的所有数据

```
hbase(main):046:0> scan 'hbase_1102'
ROW   COLUMN+CELL                                                             
 001  column=cf1:gender, timestamp=1478053832459, value=man                   
 001  column=cf1:name, timestamp=1478053787178, value=Tom                     
 001  column=cf2:chinese, timestamp=1478053848225, value=90001  column=cf2:math, timestamp=1478053858144, value=911 row(s) in0.0140seconds
```

查看其中某一个Key的数据

```
hbase(main):048:0> get'hbase_1102','001'
COLUMN                    CELL                                                                    
 cf1:gender               timestamp=1478053832459, value=man                                      
 cf1:name                 timestamp=1478053787178, value=Tom                                      
 cf2:chinese              timestamp=1478053848225, value=90                                       
 cf2:math                 timestamp=1478053858144, value=914 row(s) in0.0290seconds
```

添加一个列族

```shell
alter 'member', 'id'
```

删除一个列族

```shell
alter 'member', {NAME => 'member_id', METHOD => 'delete’}
```