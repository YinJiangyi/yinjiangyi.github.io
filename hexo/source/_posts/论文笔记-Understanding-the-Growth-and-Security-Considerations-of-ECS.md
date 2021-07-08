---
title: '论文笔记-Understanding the Growth and Security Considerations of ECS'
typora-root-url: ../../source
date: 2021-07-06 19:26:14
tags: DNS扩展
categories: 论文
sticky:
---



### Abstract

DNS的重要性要求对DNS扩展的影响做深入研究。本文研究了ECS（EDNS Client Subnet）这一扩展功能的具体影响。**该扩展已被广泛使用，但研究发现没有达到预期的用户隐私的效果**。对造成的**正面和负面的影响**进行具体介绍。

### Introduction

**贡献**

- show how the protocol has grown both before and after its official adoption in 2016. 
- 正负面影响 We discuss how it may provide more freedom to end users and aid security practitioners. At the same time, we discuss how it potentially exacerbates existing threats (i.e., DNS leaks)
- vast majority of highly ranked ECS-enabled domains *do not* benefit from the use of ECS.

### 重点

#### ECS的作用

> - CDN使用DNS获取查询IP，根据IP对用户进行地域调度。但这里获取的IP地址是DNS地址，而不是用户真实的IP地址。
> - 大多数情况下，我们假设用户通过会使用离自己网络最近的DNS resolver，CDN调度基本还是准确的。
> - 但也有很多nameserver设置错误，或者用户使用google public dns（nameserver 8.8.8.8/8.8.4.4）或opendns进行DNS resolver

比如：

> 1. 国内用户设置nameserver 8.8.8.8 (dig xxx.com @8.8.8.8)
> 2. 我们得到的DNS query IP是74.125.16.208，判断IP属于**美国,,,加利福尼亚州山景市谷歌公司**
> 3. 这个时候，我们的DNS会返回离美国加州最近的CDN节点IP给用户。
> 4. 国内用户错误的调度到美国节点…… ![:(](file:///Users/joy/MYBLOG/yinjiangyi.github.io/hexo/source/images/论文笔记-Understanding-the-Growth-and-Security-Considerations-of-ECS/aHR0cDovL25vb3BzLm1lL3dwLWluY2x1ZGVzL2ltYWdlcy9zbWlsaWVzL2ljb25fc2FkLmdpZg.gif?lastModify=1625624194?lastModify=1625625465)

#### ECS造成的影响

ECS的使用不影响below递归部分的过程，而是增加了和权威服务器之间的信息传输，携带了clientip信息（ip掩码），用于优化cdn调度。

<img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Understanding-the-Growth-and-Security-Considerations-of-ECS/image-20210707104315072.png" alt="image-20210707104315072" style="zoom:33%;" />

- 老式DNS网络拓扑：DNS和HTTP流量遵循相同的网络路径

  <img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Understanding-the-Growth-and-Security-Considerations-of-ECS/image-20210707104900225.png" alt="image-20210707104900225" style="zoom:50%;" />

- 目前的情况：递归dns解析器往往是cloud DNS或open DNS，而不是和client在同一as中的server，并且web也会讲dns服务托管到第三方平台，**所以这时使用ECS扩展时，会导致Client IP信息在本来无关的网络拓扑（红色部分）中传播。**这部分的变化其实是位于第2-7步

  <img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Understanding-the-Growth-and-Security-Considerations-of-ECS/image-20210707105317279.png" alt="image-20210707105317279" style="zoom:50%;" />

### 数据集

<img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Understanding-the-Growth-and-Security-Considerations-of-ECS/image-20210707112757294.png" alt="image-20210707112757294" style="zoom:33%;" />

#### 结论

- Measuring ECS Adoption

- Potential Security Challenges

  - 50%比例的request前缀是/24，可以通过client IP的掩码识别到所属机构

    <img src="/images/%E8%AE%BA%E6%96%87%E7%AC%94%E8%AE%B0-Understanding-the-Growth-and-Security-Considerations-of-ECS/image-20210707140727936.png" alt="image-20210707140727936" style="zoom:33%;" />

- potential security benefits

