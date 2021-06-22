---
title: >-
  论文笔记 1-08 GraphGAN: Graph Representation Learning with Generative Adversarial
  Nets
date: 2021-01-25 16:47:27
tags: 
- 论文笔记
- GAN
categories: 论文
typora-root-url: ../../source
---

Graph GAN 论文精读及相关知识补充。<!--more-->

## 信息

AAAI, 2018

## 阅读

### ABSTRACT

graph representation分为两种，生成模型和判别模型，前者学习connectivity distribution，后者预测两个节点间存在边的概率。本文提出GraphGAN。对于任意一个节点，生成器生成fake samples of its underlying true connectivity distribution，判别器判断是否为真。实验验证了链路预测、节点分类和推荐方面应用的优越性。、

### INTRODUCTION

陈述两个内容：

- 提出GraphGAN进行准确的链路预测

- softmax不适用与生成器：1）平等对待所有节点不考虑网络结构 2）time-consuming

  提出graph-softmax