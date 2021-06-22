---
title: "论文笔记 1-03 Discovering HTTPSified Phishing Websites Using the TLS Certificates Footprints"
date: 2021-01-07 18:12:38
tags: 论文笔记
categories: 论文
typora-root-url: ../../source
---

## 信息

**Discovering HTTPSified Phishing Websites Using the TLS Certificates Footprints**

EuroS&PW  2020

## 贡献

提供一种加密环境下对钓鱼网站进行预先检测的方式，对证书中CN进行聚类，提取命名模式，基于文本匹配识别可能用户钓鱼网站的证书。

## 方法

黑名单证书聚类 -> 基于类别内CN进行template提取 -> 计算template信息熵，采用大于阈值的template进行检测

<img src="/images/论文笔记 1-03/image-20210107202008077.png" alt="image-20210107202008077" style="zoom:33%;" />

 

<img src="/images/论文笔记 1-03/image-20210107201736361.png" alt="image-20210107201736361" style="zoom:50%;" />

<img src="/images/论文笔记 1-03/image-20210107201754332.png" alt="image-20210107201754332" style="zoom:50%;" />

> **Example:** 
>
> For the purpose of illustration, we present an example of template extraction process. Suppose that we obtain two CNs, **apple-accountverify123[.TLD]2** and **payment-accountverify55[.TLD]** in a cluster. The substring common to the two strings is -accountverify. The regexps of these CNs are **[a-z]{5}-accountverify[0-9]{3}[.TLD]** and **[a-z]{7}-accountverify[0-9]{2}[.TLD].**
>
> Combining the two regexps yields the following regexp, **[a-z]{5,7}-accountverify[0-9]{2,3}[.TLD].** We now calculate the *entropy reduction*. The number of characters that constitute -accountverify is 14, and the average number of characters of the regexp part La and Ld are 6 and 2.5, so the total string length L is the sum of these, 22.5. Thus, using Eqn. 1 and Eqn. 2, we obtain the following results: B(u) = 116.3, Be(u) = 36.5, and **d(e) = 79.8.** Since the entropy reduction **exceeds the threshold 55,** we **adopt the regexp as a template.** Using this template, a certificate whose CN is **google-accountverify37[.TLD] is detected** as the one used by phishing websites. However, although **security-accountverify9[.TLD] contains the same substring, our approach does not detect** it because the number of characters of the regexp is different from those of the template.

## 数据

- 公开phishing数据集获取websites黑名单 

  > We collect the phishing URLs from OpenPhish October 2018 to January 2019.

- 根据黑名单从CT log获取证书（2638），经过过滤剩下1634个证书、

- 在CT log中收集由Lets encryp和cPanel签发的证书进行分析，分析结果通过virustotal验证准确情况

结果：

- 38.7M证书中，识别1650个用户钓鱼网站的证书，1049个CN，90.8%在virustotal上至少有一个报警

## 质疑

- 没有针对钓鱼网站的特征，其他恶意是不是同样适用
- wild实验的识别对象是免费CA签发的证书，本身存在bias，template的恶意信息识别能力本质上可能并不强，但是整个逻辑合理

## 工作帮助

- 规避质疑的万能模板：

  > Note that while our approach does not aim to replace the previous defense mechanisms against phishing attacks, our experimental results indicate that our approach is an appealing complement to the conventional countermeasures against threats of phishing attacks.



## 相关文献

1. Hunting malicious tls certificates with deep neural networks
2. Beyond the lock icon: Real-time detection of phishing websites using public key certificates
3. Certified phishing: Taking a look at public key certificates of phishing websites (CCF-C)



## 背景知识

- Certificate Transparency (CT)
  https://imququ.com/post/certificate-transparency.html
  https://www.bilibili.com/read/cv3444946/

  > CT是为了防止CA或者其他恶意人员伪造服务器证书而开发的。目标是提供一个**开放的审计和监控系统**，可以让任何域名所有者或者 CA 确定证书是否被错误签发或者被恶意使用，从而提高 HTTPS 网站的安全性。CT要求CA**公开地宣布其颁发的每一个**数字证书(**将颁发记录到日志中**)。证书日志提供给用户一个查找某个给定域名颁发的所有数字证书的途径。



