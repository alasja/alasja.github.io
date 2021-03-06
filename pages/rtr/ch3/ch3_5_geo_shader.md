---
title: "3.5 几何着色器"
keywords: sample homepage
tags: [getting_started]
sidebar: rtr
layout: book
permalink: rtr_ch3_geometry.html
summary: GPU
---

### 3.5 几何着色器
2006下半年，在DX10正式版中，几何着色器才加入到流水线中。在在流水线中的位置紧挨着顶点着色器，不过它是可选的。要想使用它得支持着色模型4.0，之前的着色模型是不支持的。

几何着色器的输入数据是单独的对象和它的顶点。这里的对象指的是网格中的一个三角形、一条线段或者一个点。几何着色器可以定义额外的图元。特别是在三角形外部三个额外顶点，也可以访问线段的临近的两个顶点。见图3.6。

![图](/images/RTR3.03.06.png)

几何着色器处理一个图元，输出0个或者多个图元。输出的形式是点、线、或者三角形带。几何着色器有时候可以一次输出多个三角带。重要的是，该着色器可以啥也不输出。通过这种方式，就能通过修改顶点、添加或者删除图元来选择性的修改网格。

几何着色器输入输出都是一个对象，但是这两个并不需要时同样的类型。比如，以三角形为输入，它的顶点和图形中心点被出输出为多个点。即便输入输出类型一致，每个顶点的数据可能也完全不一样。比如，三角面的顶点每个都加上他们的法向量。跟顶点着色器一样，几何着色器输出的每个顶点都需要在齐次裁剪空间中。

几何着色器需要保证图元的输出结果顺序跟输入时一样。这对表现有影响，因为多个着色单元并行运行的话，结果表现按顺序保存。作为性能和效果的这种，着色模型4.0中有一个极限是，每次运行只能创建2014个32位数值。所以使用一片叶子来构建成千上万的叶子在几何着色器中基本是行不通的。也不推荐用简单的棋盘网格来生成精细的网格。这个阶段主要是修改输入数据或者少量的复制，而不是大规模的复制或者放大。比如，有个应用是复制6个不同的面来模拟立方体的六个不同的面，详见8.4.3。几何着色器的其他算法也会有很好的效果，比如以一个点为基础构建多个粒子，挤压边缘来进行毛皮渲染，以及在阴影算法中搜索对象边缘。图3.7展示的更多。这些会在本书后面内容逐个讨论。

![图](/images/RTR3.03.07.png)

### 3.5.1 流输出
GPU流水线的标准用法是把顶点着色器处理过的数据，送去栅格化，就是把三角形等送去像素着色器处理。数据一直都在流水线中传递，但是访问不理即时数据。流输出的想法也是着色模型4.0引入的。在顶点着色器处理顶点之后（以及可选的几何着色器），这些数据可以放入一个流中，也就是一个有序的列表，它可以被传递到栅格阶段。栅格器就可以访问它了，实际上是流水线功能关闭了，变成一个纯粹的非图形的流处理器。数据被处理后，又可以传递回流水线，因此可以迭代处理。这个功能对模拟流水或者其他一些粒子特效非常有用，详见10.7。
