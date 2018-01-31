---
title: "GPU图形渲染管线一"
keywords: gpu pipeline
tags: [gpu pipeline]
hide_sidebar: true
layout: default
summary: 图形绘制管线描述.
---

> 摘自康玉之《GPU编程与CG语言之阳春白雪下里巴人》第二章，侵删。

## 图形绘制管线描述
----
GPU 渲染流程，即“给定视点、三维物体、光源、照明模 式，和纹理等元素，如何绘制一幅二维图像”。本章内容涉及 GPU 的基本流程和 实时绘制技术的根本原理，在这些知识点之上才能延伸发展出基于 GPU 的各项 技术，所以本章的重要性怎么说都不为过。欲登高而穷目，勿筑台于浮沙!

本章首先讨论整个绘制管线(不仅仅是 GPU 绘制)所包含的不同阶段，然 后对每个阶段进行独立阐述，最后讲解 GPU 上各类缓冲器的相关知识点。在《实时计算机图形学》一书中，将图形绘制管线分为三个主要阶段:应用 程序阶段、几何阶段、光栅阶段。

### 应用程序阶段
使用高级编程语言(C、C++、JAVA 等)进行开发，主要和 CPU、内存打交道，诸如碰撞检测、场景图建立、空间八叉树更新、视锥裁剪等 经典算法都在此阶段执行。在该阶段的末端，几何体数据(顶点坐标、法向量、 纹理坐标、纹理等)通过数据总线传送到图形硬件(时间瓶颈);数据总线是一 个可以共享的通道，用于在多个设备之间传送数据;端口是在两个设备之间传送 数据的通道;带宽用来描述端口或者总线上的吞吐量，可以用每秒字节(b/s) 来度量，数据总线和端口(如加速图形端口，Accelerated Graphic Port,AGP)将 不同的功能模块“粘接”在一起。由于端口和数据总线均具有数据传输能力，因此 通常也将端口认为是数据总线(实时计算机图形学 387 页)。
### 几何阶段
主要负责顶点坐标变换、光照、裁剪、投影以及屏幕映射(实时 计算机图形学234页)，该阶段基于GPU进行运算，在该阶段的末端得到了经过 变换和投影之后的顶点坐标、颜色、以及纹理坐标(实时计算机图形学 10 页)。
### 光栅阶段
基于几何阶段的输出数据，为像素(Pixel)正确配色，以便绘制 完整图像，该阶段进行的都是单个像素的操作，每个像素的信息存储在颜色缓冲 器(color buffer 或者 frame buffer)中。
值得注意的是:光照计算属于几何阶段，因为光照计算涉及视点、光源和物 体的世界坐标，所以通常放在世界坐标系中进行计算;而雾化以及涉及物体透明 度的计算属于光栅化阶段，因为上述两种计算都需要深度值信息(Z 值)，而深度值是在几何阶段中计算，并传递到光栅阶段的。

下面具体阐述从几何阶段到光栅化阶段的详细流程。

## 2.1 几何阶段
----
几何阶段的主要工作是“变换三维顶点坐标”和“光照计算”，显卡信息中通常 会有一个标示为“T&L”硬件部分，所谓“T&L”即 Transform & Lighting。那么为什 么要对三维顶点进行坐标空间变换?或者说，对三维顶点进行坐标空间变换有什 么用?为了解释这个问题，我先引用一段文献【3】中的一段叙述:
> Because, your application supplies the geometric data as a collection of vertices, but the resulting image typically represents what an observer or camera would see from a particular vantage point.
As the geometric data flows through the pipeline, the GPU’s vertex processor transforms the continuant vertices into one or more different coordinate system, each of which serves a particular purpose. CG vertex programs provide a way for you to program these transformations yourself.

上述英文意思是:
> 输入到计算机中的是一系列三维坐标点，但是我们最终需 要看到的是，从视点出发观察到的特定点(这句话可以这样理解，三维坐标点， 要使之显示在二维的屏幕上)。一般情况下，GPU 帮我们自动完成了这个转换。 基于 GPU 的顶点程序为开发人员提供了控制顶点坐标空间转换的方法。

一定要牢记，显示屏是二维的，GPU 所需要做的是将三维的数据，绘制到 二维屏幕上，并到达“跃然纸面”的效果。顶点变换中的每个过程都是为了这个目 的而存在，为了让二维的画面看起具有三维立体感，为了让二维的画面看起来“跃 然纸面”。
根据顶点坐标变换的先后顺序，主要有如下几个坐标空间，或者说坐标类型: Object space，模型坐标空间;World space，世界坐标系空间;Eye space，观察坐 标空间;Clip and Project space，屏幕坐标空间。图 3 表述了 GPU 的整个处理流 程，其中茶色区域所展示的就是顶点坐标空间的变换流程。大家从中只需得到一 个大概的流程顺序即可，下面将详细阐述空间变换的每个独立阶段。


 ![图3](http://img.blog.csdn.net/20170717131022336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkRhbGFzamE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2.1.1 从 object space 到 world space
> When an artist creates a 3D model of an object, the artist selects a convenient orientation and position with which to place the model’s continent vertices.
The object space for one object may have no relationship to the object space of another object.【3】

上述语句表示了 object space 的两层核心含义:
其一，object space coordinate 就是模型文件中的顶点值，这些值是在模型建模时得到的，例如，用 3DMAX 建 立一个球体模型并导出为.max 文件，这个文件中包含的数据就是 object space coordinate;其二，object space coordinate 与其他物体没有任何参照关系，注意， 这个概念非常重要，它是将 object space coordinate 和 world space coordinate 区分 开来的关键。无论在现实世界，还是在计算机的虚拟空间中，物体都必须和一个 固定的坐标原点进行参照才能确定自己所在的位置，这是 world space coordinate 的实际意义所在。

毫无疑问，我们将一个模型导入计算机后，就应该给它一个相对于坐标原点的位置，那么这个位置就是 world space coordinate，从 object space coordinate 到 world space coordinate 的变换过程由一个四阶矩阵控制，通常称之为 world matrix。

光照计算通常是在 world coordinate space(世界坐标空间)中进行的，这也 符合人类的生活常识。当然，也可以在 eye coordinate space 中得到相同的光照效 果，因为，在同一观察空间中物体之间的相对关系是保存不变的。

需要高度注意的是:顶点法向量在模型文件中属于 object space，在 GPU 的 顶点程序中必须将法向量转换到 world space 中才能使用，如同必须将顶点坐标 从 object space 转换到 world space 中一样，但两者的转换矩阵是不同的，准确的 说，法向量从 object space 到 world space 的转换矩阵是 world matrix 的转置矩阵 的逆矩阵(许多人在顶点程序中会将两者的转换矩阵当作同一个，结果会出现难 以查找的错误)。(参阅潘李亮的 3D 变换中法向量变换矩阵的推导一文)

可以阅读电子工业出版社的《计算机图形学(第二版)》第 11 章，进一步了 解三维顶点变换具体的计算方法，如果对矩阵运算感到陌生，则有必要复习一下 线性代数。

### 2.1.2 从 world space 到 eye space
每个人都是从各自的视点出发观察这个世界，无论是主观世界还是客观世 界。同样，在计算机中每次只能从唯一的视角出发渲染物体。在游戏中，都会提 供视点漫游的功能，屏幕显示的内容随着视点的变化而变化。这是因为 GPU 将 物体顶点坐标从 world space 转换到了 eye space。

所谓 eye space，即以 camera(视点或相机)为原点，由视线方向、视角和 远近平面，共同组成一个梯形体的三维空间，称之为 viewing frustum(视锥)， 如图 4 所示。近平面，是梯形体较小的矩形面，作为投影平面，远平面是梯形体 较大的矩形，在这个梯形体中的所有顶点数据是可见的，而超出这个梯形体之外 的场景数据，会被视点去除(Frustum Culling，也称之为视锥裁剪)。

![图4](http://img.blog.csdn.net/20170717131315290?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkRhbGFzamE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2.1.3 从 eye space 到 project and clip space
> Once positions are in eye space, the next step is to determine what positions are
actually viewable in the image you eventually intend trend.【3】

即:一旦顶点坐标被转换到 eye space 中，就需要判断哪些点是视点可见的。 位于 viewing frustum 梯形体以内的顶点，被认定为可见，而超出这个梯形体之外 的场景数据，会被视点去除(Frustum Culling，也称之为视锥裁剪)。这一步通常 称之为“clip(裁剪)”，识别指定区域内或区域外的图形部分的过程称之为裁剪 算法。

很多人在理解该步骤时存在一个混乱，即“不清楚裁减与投影的关系和两者 发生的先后顺序”，不少人觉得是“先裁减再投影”，不过事实并非如此。因为在 不规则的体(viewing frustum)中进行裁剪并非易事，所以经过图形学前辈们的 精心分析，裁剪被安排到一个单位立方体中进行，该立方体的对角顶点分别是 (-1,-1,-1)和(1,1,1)，通常称这个单位立方体为规范立方体(Canonical view volume, CVV)(实时计算机图形学第 9 页)。CVV 的近平面(梯形体较小的矩形面)的 X、Y 坐标对应屏幕像素坐标(左下角是 0、0)，Z 坐标则是代表画面像素深度。

多边形裁剪就是 CVV 中完成的。所以，从视点坐标空间到屏幕坐标空间 (screen coordinate space)事实上是由三步组成:
1. 用透视变换矩阵把顶点从视锥体中变换到裁剪空间的 CVV 中;
2. 在 CVV 进行图元裁剪;
3. 屏幕映射:将经过前述过程得到的坐标映射到屏幕坐标系上。

在这里，我们尤其要注意第一个步骤，即把顶点从 viewing frustum 变换到 CVV 中，这个过程才是我们常说或者听说的“投影”。主要的投影方法有两种: 正投影(也称平行投影)和透视投影。由于投影投影更加符合人类的视觉习惯， 所以在附录中会详细讲解透视投影矩阵的推导过程，有兴趣的朋友可以查阅潘宏 (网名 Twinsen)的“透视投影变换推导”一文。更详细全面的投影算法可以近一 步阅读《计算机图形学(第二版)》第 12 章第 3 节。

确定只有当图元完全或部分的存在于视锥内部时，才需要将其光栅化。当一 个图元完全位于视体(此时视体以及变换为 CVV)内部时，它可以直接进入下 一个阶段;完全在视体外部的图元，将被剔除;对于部分位于视体内的图元进行 裁减处理。详细的裁剪算法可以近一步阅读《计算机图形学(第二版)》第 12 章第5节。

附 1:透视投影矩阵的推导过程，建议阅读潘宏(网名 Twinsen)的“透视投 影变换推导”一文。
附 2:视点去除，不但可以在 GPU 中进行，也可以使用高级语言(C\C++) 在 CPU 上实现。使用高级语言实现时，如果一个场景实体完全不在视锥中，则 该实体的网格数据不必传入 GPU，如果一个场景实体部分或完全在视锥中，则 该实体网格数据传入 GPU 中。所以如果在高级语言中已经进行了视点去除，那 么可以极大的减去 GPU 的负担。使用 C++进行视锥裁剪的算法可以参阅 OGRE (Object-Oriented Graphics Rendering Engine，面向对象的图形渲染引擎)的源码。
