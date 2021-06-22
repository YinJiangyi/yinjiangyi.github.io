---
title: 李宏毅课程-GAN
date: 2021-01-22 17:21:26
tags: 
- 课程笔记
- 李宏毅
- GAN
catagories:
- DeepLearning
typora-root-url: ../../source
---

李宏毅GAN课程笔记<!--more-->

Yann LeCun:

> AN is the cool things since sliced bread
>
> 近十年来ML领域最有趣的研究

### basic idea of GAN

- generation：输入random vector，输出生成的图片、句子

  <img src="/images/李宏毅课程-GAN/image-20210122172925761.png" alt="image-20210122172925761" style="zoom:50%;" />

  Condition generation：输入的vector不是random的，而是熟悉的，比如指定类别、风格等。

- basic idea of GAN

  - generator

    <img src="/images/李宏毅课程-GAN/image-20210122173342874.png" alt="image-20210122173342874" style="zoom:50%;" />

    不同维度向量值对应不同属性的控制。

  - discrimiminator

    输出分数，表示生成内容的质量。

    <img src="/images/李宏毅课程-GAN/image-20210122175821862.png" alt="image-20210122175821862" style="zoom:50%;" />

  - generator & discriminator

    **generator为什么不自己学，discriminator存在的作用是什么？**

- algorithm

  - 初始化G和D两个network的参数
  - 每一轮：
    - fix G, update D：G生成一组对象，从database抽取一组对象，D通过两组对象进行调整参数。如何调整参数：训练参数使生成的对象分低，抽样的对象分高
    - fix D, update G：调整G的参数，使生成的对象被D打高分。（实际上相当于把G和D作为整个网络，只训练G部分的参数，使评分尽量高。）