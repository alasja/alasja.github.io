---
title: "符号及定义"
keywords: sample homepage
tags: [getting_started]
sidebar: rtr
layout: book
permalink: rtr_ch1_notation.html
summary: RTR第一章1.2，各个符号的描述及一些定义的讲解。
---

首先，我们得解释下本书用到的数学符号，更详尽的解释请查看附录A。

## 数学符号
表1.1列出了我们会用到的大部分数学符号。部分概念我们会在后面详细说明。

![table1.1](/images/table1_1.png)

角度跟缩放使用$R$，他们是实数。向量和点表示成小写粗体字母，表示成这样：

$$
v = \begin{pmatrix} v_x \\ v_y \\ v_z \end{pmatrix}
$$

这被称为列向量模式，是现在计算机图形学(一下简称CG)领域通用的方式。有些时候因为易读的原因，我们会用$(v_x, v_y, v_z)$来代替正式的$(v_x, v_y, v_z)^T$形式。

在齐次坐标中(详见附录A.4)，坐标表示成$v = (v_x\ v_y\ v_z\ v_w)^T$， 向量表示为$v = (v_x\ v_y\ v_z\ 0)^T$，坐标点则是$v = (v_x\ v_y\ v_z\ 1)^T$。有时候我们只用三个元素来表示向量和点，但是我们尽可能统一各种用法。对矩阵操作来说，统一向量和点的符号非常有用(参考第四章变换)。

