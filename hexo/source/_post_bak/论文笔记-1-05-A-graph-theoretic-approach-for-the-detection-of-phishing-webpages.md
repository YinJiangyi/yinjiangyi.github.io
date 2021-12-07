---
title: 论文笔记 1-05 A graph-theoretic approach for the detection of phishing webpages
date: 2021-01-13 16:57:51
tags: 论文笔记
categories: 论文
typora-root-url: ../../source
---

## 信息

**A graph-theoretic approach for the detection of phishing webpages**

Computers & Security 2020  CCF-B

## 借鉴

- baseline 算法：19年CCF-B

  **A new hybrid ensemble feature selection framework for machine learning-based phishing detection system**
  
- introduction里可以添加相关report佐证直观事实

- <font color=#CD6155>**冷启动、新项目**</font>问题的解决

  现有方法：

  - 网页内容（攻击者通过转义和无关插入规避）
  - 第三方服务（稳定性和一致性问题：whois查找服务有可能返回查询网站的不完整信息，造成不稳定。证伪论文：[Fuzzy rough set feature selection to enhance phishing attack detection](https://ieeexplore.ieee.org/abstract/document/8858884/)）

- 考虑把特征的justification集成到复杂图模型中学习

## 不足

数据集小，效果不明显

## 阅读

### 摘要

- 基于图论提出新的图特征以提高url探测效果，主要贡献在于特征发现，而不在算法
- 方法。提取网页上的超链接作为邻居网页，构造网络结构。计算相关图度量作为分类特征。提取的特征考虑了钓鱼网站的生存模式

> we capitalise on concepts from graph theories, and propose a set of novel graph features to improve the phishing detection accuracy.
>
> Unlike conventional features, the proposed graph features leverage inherent phishing patterns that are only visible at a higher level of abstraction, thus making it robust and difficult to be evaded by direct manipulations on the webpage contents. 

### Introduction

- phishing网站活动具有季节周期特性，受黑五等流量峰值情况影响

- Widespread adoption of HTTPS makes it more difficult.

- 黑名单技术对新的钓鱼网站无效

- zero-day phishing问题没有较好的办法，需要effective and time-invariant的方法

- => 采用图的方法，构造ego network, 挖掘deeper level feature

  > <font color=#CD6155>**To the best of our knowledge,**</font> graph features are yet to be fully utilised for phishing detection.
  >
  > <img src="/images/论文笔记-1-05-A-graph-theoretic-approach-for-the-detection-of-phishing-webpages/image-20210113175747561.png" alt="image-20210113175747561" style="zoom:67%;" />

### Related works

- 网页内容特征

- hyperlink features

  > The new features were formulated based on the following rationales: 
  > (a) Phishers often modify hyperlinks in the phishing webpage to point to a single common URL in the phishing domain, and;
  > (b) Legitimate login webpages contain at least one hyperlink for logical navigation purposes such as login, account registration, password reset, etc. 

- Deep learning

- ……

### Methodology

- pilot study

  - believability
  - Capability to evade phishing detectors
  - Cost of implementation

  ![image-20210114095631296](/images/论文笔记-1-05-A-graph-theoretic-approach-for-the-detection-of-phishing-webpages/image-20210114095631296.png)

  - 钓鱼网站为了提高可信度，会在网页中和插入目标合法网站相同的超链接。但是合法网站中的超链接往往是内部url，因此可以通过内部url占所有超链接的比例判断是否为钓鱼网站。（已被广泛使用，因此攻击者选择通过牺牲可信度绕过检测）
  - 绕过检测的方法。把外部超链接改为内部或broken hyperlinks
  - 钓鱼网站生存周期短，所以尽可能降低实现成本。从合法网站快速拷贝源代码，过程中会保留原始超链接，反过来有增强了

- proposed architecture

  <img src="/images/论文笔记-1-05-A-graph-theoretic-approach-for-the-detection-of-phishing-webpages/image-20210114105904527.png" alt="image-20210114105904527" style="zoom: 67%;" />

  <img src="/images/论文笔记-1-05-A-graph-theoretic-approach-for-the-detection-of-phishing-webpages/image-20210114105923604.png" alt="image-20210114105923604" style="zoom: 67%;" />

  网络构造过程：取query webpage和他们的两跳邻居

  > 1) Vertex on L0 is labelled red. 2) Vertices on L1 and L2 with the same domain name as the query webpage are labelled red. 3) The remaining vertices on L1 are labelled blue. 4) The remaining vertices on L2 are labelled green. 

- Proposed graph features

  - 各色节点百分比，非query website的红色节点百分比
  - 非query website的红色节点∩broken数量
  - Query website的出入度，去回环的出入度，中心度
  - 图密度，红色节点子图密度
  - edge_betweenness_centrality_mean
  - is_semiconnected 
  - num_strongly_connected_components 
  - num_attracting_components 

### 数据

黑名单获得500个钓鱼网站，500个合法网站url，下载2018年9月-12月的website（python GNU wget 下载工具）。

### 实验

<img src="/images/论文笔记-1-05-A-graph-theoretic-approach-for-the-detection-of-phishing-webpages/image-20210114113456836.png" alt="image-20210114113456836" style="zoom:50%;" /> <img src="/images/论文笔记-1-05-A-graph-theoretic-approach-for-the-detection-of-phishing-webpages/image-20210114113545520.png" alt="image-20210114113545520" style="zoom:50%;" />

