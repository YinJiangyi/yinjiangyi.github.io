---
title: SSL/TLS会话的流量分析
date: 2021-01-18 18:04:59
tags: HTTPS
Categories: 计算机网络
---

`待完成`

介绍SSL/TLS协议相关理论基础，并对真实会话流量进行分析。<!--more-->

参考：
https://zhuanlan.zhihu.com/p/133375078

## 理论知识梳理

- SNI & SAN

  - SNI: Server name indication **服务器名称指示**，字段在握手包的Extension中（[关于extension中的常用字段介绍](https://halfrost.com/https-extensions/)），在握手过程开始时（client hello）通过客户端告诉它正在连接的服务器的主机名称。这时候SSL连接还未建立起来，因此SNI信息是明文传输的。

    实际会带来的后果并不涉及安全问题，仅仅只是所请求的服务器泄露出去了，连接可能受到干扰。如果**网络防火墙的包过滤系统检查SNI信息的话，就可以屏蔽某些域名的https访问，就像屏蔽http一样简单。**

  - SAN: subject alternative name **主题别名**，数字证书中的字段，表示证书能够被多个域名所使用。客户端通过SNI指示要连接到哪个主机名。

