---
title: 论文笔记 1-06 Multi-Class Imbalanced Graph Convolutional Network Learning
date: 2021-01-15 17:49:49
tags: 论文笔记
categories: 论文
typora-root-url: ../../source
---

## 信息

**Multi-Class Imbalanced Graph Convolutional Network Learning**

IJCAI 2020  CCF-A

## 借鉴

<!--more-->

## 阅读

### 摘要

针对多分类中标签不均衡问题，提出双重正则GCN（Dual-Regularized）进行节点表达学习。提出 class-conditioned adversarial training process。实验结果表明在节点分类、图聚类和可视化上都表现优秀。

### Introduction

- 不平衡问题的存在

> **Existing methods often** assume that the input class distributions are nearly or perfectly balanced, *i.e.*, balanced label samples for each class are deliberately provided to ensure representation learning equilibrium across multiple classes thereby **avoiding the class imbalance problem entirely**.

> When generalizing to graphs with an imbalanced class distribution, existing **GCN methods have a tendency to overfit to majority classes**, resulting in undesirable embedding re- sults for the minority classes.

- 不平衡问题在基于图的解决方法中较难解决（简单的重采样等不适用）

> To the best of our knowledge, this is the first work that studies the node-level class-imbalanced graph embedding problem with graph neural networks.

