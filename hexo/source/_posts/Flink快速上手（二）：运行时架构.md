---
title: Flink快速上手（二）：运行时架构
date: 2021-01-28 15:09:04
tags: Flink
catageries: 大数据
typora-root-url: ../../source
---

介绍Flink运行时组件，任务提交流程和任务调度原理。<!--more-->

### Flink运行时组件

四大组件

- jobManager：作业管理器（负责调度）
  - 主进程，每个flink应用程序都会被一个不同的jobmanager控制执行。
  - jobManager先接收到要执行的应用程序，这个应用程序包括：**作业图jobGraph**，**逻辑数据流图**和打包了所有的类、库和其他资源的**jar包**
  - 然后JM将JobGraph 转换成一个物理层面的数据流图，这个图被叫做**执行图(ExecutionGraph)**，执行图包含了所有可以并发执行的任务。明确说明了程序应该被如何执行。
  - 另外JM回向ResourceManager**请求必要的资源**（slot），运行过程中JM也负责所有需要中央协调的操作，比如checkpoint的协调。
- taskManager：任务管理器（负责干活）
  - Flink 中的工作进程。通常在 Flink 中会有多个 TaskManager 运行，每一个 TaskManager都包含了一定数量的插槽(slots)。插槽的数量限制了 TaskManager 能够执行的任务数量。
  - 启动之后，TaskManager 会向资源管理器**注册它的插槽**；收到资源管理器的指令后， TaskManager 就会将一个或者多个插槽提供给 JobManager 调用。JobManager 就可以向插槽分配任务(tasks)来执行了。
  - 在执行过程中，一个 TaskManager 可以跟其它运行同一应用程 序的 TaskManager **交换数据**（不同阶段之间的输入输出等）。
- resourceManager：资源管理器（调配slot）
  - 主要负责管理任务管理器(TaskManager)的插槽(slot)，TaskManger 插槽是 Flink 中 定义的**处理资源单元**。
  - 当 JobManager 申请插槽资源时，ResourceManager 会将有空闲插槽的TaskManager分配给JobManager。如果ResourceManager没有足够的插槽 来满足 JobManager 的请求，它还可以向资源提供平台（YARN, Mesos, k8s）发起会话，以提供启动 TaskManager 进程的容器。
- dispacher：分发器
  - 它为应用提交提供了 REST 接口。当一个应用被提交执行时，分发器 就会启动并将应用移交给一个 JobManager。
  - Dispatcher 也会启动一个 Web UI，用来方便地展示和监控作业执行的信息。Dispatcher 在架构中可能并不是必需的，这取决于应 用提交运行的方式。

### 任务提交流程

<img src="/images/Flink快速上手（二）：运行时架构/image-20210128152505563.png" alt="image-20210128152505563" style="zoom:80%;" />

* 资源分配平台模式下，需要多少资源就启动多少资源

<img src="/images/Flink快速上手（二）：运行时架构/image-20210128152930628.png" alt="image-20210128152930628" style="zoom:80%;" />

### 任务调度原理

<img src="/images/Flink快速上手（二）：运行时架构/image-20210128153540447.png" alt="image-20210128153540447" style="zoom:80%;" />

- 三个问题？
  - 如何实现并行计算：每个任务设置并行度，实现并行任务，分配给不同的slot
  - 并行的任务需要占用多少slot：并行度最高的slot数是任务占用的slot数
  - 一个流处理程序，包含多少个任务，哪些计算操作可以合并

#### TaskManager、slot和paralelism的关系

- TaskManager简单理解就是应用进程中真正干活的机器，每一个TM都是一个JVM进程。在conf/workers中配置了几台，TM就有几个。
- Slot是TM划分的固定大小的子集，多个slot用来实现多线程并行任务。如果一个TM有3个slot，那么它会将内存分成三份。不同slot之间只用来隔离task的受管理的内存，不涉及到CPU的隔离，因为会存在一个TM上只有一个CPU，但是划分了多个slot的情况。
- Paralelism是为应用程序（特定子任务）设置的并行度。和slot的区别是，slot是划分好的分区个数，而paralelism是实际使用的个数。slot的对象是TM，是静态值，Paralelism设置的对象是应用程序，是动态概念。

如果一个flink集群有2个TM，slot设置为3，那么每个TM可以接收3个task，如果设置默认并行度为1，那么就有5个slot是空闲的。

#### 并行任务配置执行规则

##### paralelism配置**优先级**

1. 代码中`.setParallelism(value)` 为子任务设置并行度
2. 如果代码中没有设置，取提交job执行程序时`-p`参数 `./flink run -c com.atguigu.wc.StreamWordCount –p 2`， 或web interface中的paralelism值
3. 如果1和2都没有设置，采用conf/flink-conf.yaml中的默认设置

##### 子任务共享slot

1. 同一个子任务的不同并行任务必须分配到不同的slot中。

2. 如果没有特别设置，具**有先后执行顺序的子任务会共享同一个slot**。如下图，source和map被合并为同一个任务了（合并任务策略不详），和后续的子任务keyby默认会共享slot，这样做的原因是：

   <img src="/images/Flink快速上手（二）：运行时架构/image-20210128170925680.png" alt="image-20210128170925680" style="zoom: 50%;" />

   1）虽然不同子任务的并行度可能不同（sink的并行度为1），但是可以保证至少在一个slot中形成一个任务的pipline，提高**健壮性**，

   2）如果不共享，如下图，不同子任务消耗资源能力不同，导致**资源分配不均**。（sink较keyby分配的slot空闲的多）

   <img src="/images/Flink快速上手（二）：运行时架构/image-20210128171341054.png" alt="image-20210128171341054" style="zoom: 50%;" />

3. 如果需要强制某两个slot不在同一个slot中，通过`.slotSharingGroup("groupname")` 设置共享slot的子任务组。比如把sink设为”red“，keyby设为”green“，那么就不存在共享行为。

4. 一般情况下，即没有设置slot共享分组时，job所占用的slot个数和最大并行度的子任务设置的并行度一致。如下图，该job共有16个子任务，但是占用slot为4个，因为默认子任务间slot共享。

   <img src="/images/Flink快速上手（二）：运行时架构/image-20210128172041077.png" alt="image-20210128172041077" style="zoom: 50%;" />

##### 流处理程序分析和执行流程

- 程序会被映射为**逻辑数据流**：包括三大块：Source、transform、sink，每一个Dataflow以一个或多个sources开始以一个或多个sinks结束。类似任意的**有向无环图（DAG）**

- Streamgraph(客户端上的图) -> jobGraph(合并任务) -> ExecutionGraph(JM做转化) -> 物理执行图

  <img src="/images/Flink快速上手（二）：运行时架构/image-20210128173439856.png" alt="image-20210128173439856" style="zoom:80%;" />

  - client上生成StreamGraph，其中keyby只是分区，所以不作为一个任务，而是map之后直接进行按key的聚合计算（sum）
  - JobGraph生成过程中，把符合条件的过程合并（**什么条件？**）
  - 客户端把jobgraph连同jar包提交给JM，JM拿到后拆成和执行的并行化版本ExecutionGraph
  - 可执行图传给TM，TM得到物理执行图做计算

##### JobGraph生成过程中子任务如何合并？

- 算子间的数据传输形式

  - One-to-one：可以维护分区及元素的顺序，如map、filter、filtermap，类似于 spark 中的**窄依赖**
  - redistrbution：存在重分区过程，eyBy() 基于 hashCode 重分区、broadcast 和 rebalance(轮询) 会随机重新分区，这些算子都会引起 redistribute 过程，而 redistribute 过程就类似于 Spark 中的 shuffle 过程。类似于 spark 中的**宽依赖**

  ![image-20210128174940908](/images/Flink快速上手（二）：运行时架构/image-20210128174940908.png)

- 任务链（opetator chains）

  在特定条件下为减少本地通信的开销，**将满足条件的两个或多个算子通过本地转发的方式进行连接**。即，满足**并行度相同的one-to-one操作的两个相连的算子**可以连在一起作为一个task。如下图中，满足合并条件的为keyagg和sink。

<img src="/images/Flink快速上手（二）：运行时架构/image-20210128180545901.png" alt="image-20210128180545901" style="zoom:67%;" />

- 扩展：
  - 如何不合并：设置共享组，把key agg和sink设置到两个共享组
  - 如果设置共享组就不能共享slot了，如何共享slot的情况下不合并：
    - 加一个重分区操作 `.shuffle()` `.rebalance()`
    - 加 `.disableChaining()` 设置和前后都不参与任务链合并
    - 也可以全局设置 `env.disableChaining()`
    - 开始新任务链合并 `.startNewChain()`

