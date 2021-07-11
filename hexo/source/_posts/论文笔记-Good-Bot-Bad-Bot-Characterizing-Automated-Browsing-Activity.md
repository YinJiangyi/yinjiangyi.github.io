---
title: '论文笔记-Good Bot, Bad Bot: Characterizing Automated Browsing Activity'
typora-root-url: ../../source
date: 2021-07-06 19:26:14
tags: bot
categories: 论文
sticky:
---

### 摘要

目标是恶意bot测量和分析，通过部署大量（100个）蜜罐站点（honeysite），在28w的IP中发现7w的恶意bots，其中80%以上的bots声称是来自firefox和chrome。

### Intorduction

根据行为区分恶意和非恶意

思路：

-> 恶意bot占大量流量
-> 现有方法陷入获取准确标注数据集来训练标注模型的怪圈
-> 提出基于honeysite的方法绕过对真实用户和bot进行区别分析的过程

流程：

-> 注册从未使用过且没有推广过的域名，使确保对其进行访问的都是bot
-> 分析收到的请求流量，57%为明显恶意；47,667 bots向login界面发送未经要求的post请求；12,183 bots进行web-app网络指纹；有些在漏洞暴露当天就进行针对性的攻击
-> 86%以上的进行身份欺骗——判断方法：声称身份和tls或http头生成的指纹不匹配 

### Background

（跳过）

- Bots and automated browsing.
- Browser fingerprinting.
- TLS fingerprinting. 
- Behavioral analysis.
- Browsing sessions.

### System design

 Aristaeus包括三个部分: honeysites, log aggregation, and analysis modules. 

<img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Good-Bot-Bad-Bot-Characterizing-Automated-Browsing-Activity/image-20210706195410678.png" alt="image-20210706195410678" style="zoom: 25%;" />

#### HoneySite

- 确保所有流量都非正常真实用户流量
- 确保后期能够将同一bot使用的不同的IP的请求关联一起，每个honeysites具备多个层级的指纹生成功能: 
  *browser fingerprinting*, *behavior fingerprinting*, and *TLS fingerprinting*. 

### Traffic 分析

- 访问来源

<img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Good-Bot-Bad-Bot-Characterizing-Automated-Browsing-Activity/image-20210706201404523.png" alt="image-20210706201404523" style="zoom:33%;" />

- 端点访问分布：爬虫具有访问特异性，login页面比较受欢迎

  <img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Good-Bot-Bad-Bot-Characterizing-Automated-Browsing-Activity/image-20210706201832200.png" alt="image-20210706201832200" style="zoom:33%;" />

- bot类别

  <img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Good-Bot-Bad-Bot-Characterizing-Automated-Browsing-Activity/image-20210706202222376.png" alt="image-20210706202222376" style="zoom:33%;" />

