---
title: >-
  论文笔记 1-07 An End-to-End Large-Scale Measurement of DNS-over-Encryption: How
  Far Have We Come?
date: 2021-01-18 16:12:43
tags: 论文笔记
categories: 论文
typora-root-url: ../../source
---

`待精读`

DNS加密协议发展及部署现状。IMC 2019最佳论文提名

从服务端、客户端、网络传输三个方面探讨DoE生态的发表现状<!--more-->

## 信息

**An End-to-End Large-Scale Measurement of DNS-over-Encryption: How Far Have We Come?**

## 阅读

DoT：在用户发起请求之前，和DNS服务器协商建立TLS会话，端口853

DoH：将DNS包嵌入到HTTP消息内容中，和HTTP共享使用443端口

<img src="/images/论文笔记-1-07-An-End-to-End-Large-Scale-Measurement-of-DNS-over-Encryption-How-Far-Have-We-Come/image-20210118162114811.png" alt="image-20210118162114811" style="zoom: 50%;" />

- DoE服务器探测：

  - DoT可以通过853端口扫描进行探测

  - DoH和http服务共享443端口，不能通过端口扫描。但是RFC文档和大型服务提供商都采用URL模板提供访问（如:/dnd-query），可以在大规模的url数据集中发现DoH服务器

    <img src="/images/论文笔记-1-07-An-End-to-End-Large-Scale-Measurement-of-DNS-over-Encryption-How-Far-Have-We-Come/image-20210118163513719.png" alt="image-20210118163513719" style="zoom:50%;" />

