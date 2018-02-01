---
title: "GPU图形渲染管线二"
keywords: gpu pipeline
tags: [gpu pipeline]
hide_sidebar: true
layout: post
permalink: /other_:title.html
summary: 图形绘制管线描述.
---

> 摘自康玉之《GPU编程与CG语言之阳春白雪下里巴人》第二章，侵删。

## 2.2 Primitive Assembly && Triangle setup
Primitive Assembly，图元装配，即将顶点根据 primitive(原始的连接关系),还原出网格结构。网格由顶点和索引组成，在之前的流水线中是对顶点的处理， 在这个阶段是根据索引将顶点链接在一起，组成线、面单元。之后就是对超出屏 幕外的三角形进行裁剪，想象一下:一个三角形其中一个顶点在画面外，另外两 个顶点在画面内，这是我们在屏幕上看到的就是一个四边形。然后将该四边形切 成两个小的三角形。

此外还有一个操作涉及到三角形的顶点顺序(其实也就是三角形的法向量朝 向)，根据右手定则来决定三角面片的法向量，如果该法向量朝向视点(法向量 与到视点的方向的点积为正)，则该面是正面。一般是顶点按照逆时针排列。如 果该面是反面，则进行背面去除操作(Back-face Culling)。在 OpenGL 中有专门 的函数 enable 和 disable 背面去除操作。所有的裁剪剔除计算都是为了减少需要 绘制的顶点个数。

附:在 2.2 和 2.3 节都提到了裁减的概念，实际裁减是一个较大的概念，为 了减少需要绘制的顶点个数，而识别指定区域内或区域外的图形部分的算法都称 之为裁减。裁减算法主要包括:视域剔除(View Frustum Culling)、背面剔除 (Back-Face Culling)、遮挡剔除(Occlusing Culling)和视口裁减等。

处理三角形的过程被称为 Triangle Setup。到目前位置，我们得到了一堆在 屏幕坐标上的三角面片，这些面片是用于做光栅化的(Rasterizing)。


## 2.3 光栅化阶段
###2.3.1 Rasterization
光栅化:决定哪些像素被集合图元覆盖的过程(Rasterization is the process of determining the set of pixels covered by a geometric primitive)。经过上面诸多坐标 转换之后，现在我们得到了每个点的屏幕坐标值(Screen coordinate)，也知道我 们需要绘制的图元(点、线、面)。但此时还存在两个问题，

问题一:点的屏幕坐标值是浮点数，但像素都是由整数点来表示的，如果确 定屏幕坐标值所对应的像素?
问题二:在屏幕上需要绘制的有点、线、面，如何根据两个已经确定位置的 2 个像素点绘制一条线段，如果根据已经确定了位置的 3 个像素点绘制一个三角 形面片?

首先回答一下问题一，“绘制的位置只能接近两指定端点间的实际线段位置， 例如，一条线段的位置是(10.48，20.51)，转换为像素位置则是(10，21)”(计 算机图形学(第二版)52 页)。
对于问题二涉及到具体的画线算法，以及区域图元填充算法。通常的画线算 法有 DDA 算法、Bresenham 画线算法;区域图元填充算法有，扫描线多边形填 充算法、边界填充算法等，具体请参阅《计算机图形学(第二版)》第 3 章。

这个过程结束之后，顶点(vertex)以及绘制图元(线、面)已经对应到像素 (pixel)。下面阐述的是“如何处理像素，即:给像素赋予颜色值”。


### 2.3.2 Pixel Operation
Pixel operation 又称为 Raster Operation(在文献【2】中是使用 Raster Operation)，是在更新帧缓存之前，执行最后一系列针对每个片段的操作，其目 的是:计算出每个像素的颜色值。在这个阶段，被遮挡面通过一个被称为深度测 试的过程而消除，这其中包含了很多种计算颜色的方法以及技术。Pixel operation 包含哪些事情呢?

1. 消除遮挡面
2. Texture operation，纹理操作，也就是根据像素的纹理坐标，查询对应的纹理值; 
3. Blending 混色，根据目前已经画好的颜色，与正在计算的颜色的透明度(Alpha)， 混合为两种颜色，作为新的颜色输出。通常称之为 alpha 混合技术。 当在屏幕 上绘制某个物体时，与每个像素都相关联的哟一个 RGB 颜色值和一个 Z 缓冲器 深度值，另外一个称为是 alpha 值，可以根据需要生成并存储，用来描述给定像 素处的物体透明度。如果 alpha 值为 1.0，则表示物体不透明;如果值为 0，表示 该物体是透明的，

	从绘制管线得到一个 RGBA，使用 over 操作符将该值与原像素颜色值进行 混合，公式如下:
![公式1](http://img.blog.csdn.net/20170717132403235?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkRhbGFzamE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
	a是透明度值(alpha)，ca 表示透明物体的颜色，cs 表示混合前像素的颜色
值， cd 是最终计算得到的颜色值。Over 操作可以用于照片混合和物体合成绘制 方面，这个过程称为合成(compositing)。可以联想一下，OGRE 中有一种技术称为 compositor(合成器)。

	此外还需要提醒的一点是:为了在场景中绘制透明物体，通常需要对物体进 行排序。首先，绘制不透明的物体;然后，在不透明物体的上方，对透明物体按 照由后到前的顺序进行混合处理。如果按照任意顺序进行混合，那么会产生严重 的失真。既然需要排序，那么就需要用到 z buffer。关于透明度、合成的相关知 识点，可以在《实时计算机图形学(第二版)》第四章 4.5 节(59 页)中得到更 多详尽的知识。

4. Filtering，将正在算的颜色经过某种 Filtering(滤波或者滤镜)后输出。 可以理解为:经过一种数学运算后变成新的颜色值。

该阶段之后，像素的颜色值被写入帧缓存中。图 5 来自文献【2】1.2.3，说 明了像素操作的流程
![图5](http://img.blog.csdn.net/20170717132544860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkRhbGFzamE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 2.4 图形硬件

这一节中主要阐述图形硬件的相关知识，主要包括 GPU 中数据的存放硬件， 以及各类缓冲区的具体含义和用途，如:z buffer(深度缓冲区)、stencil buffer (模板缓冲区)、frame buffer(帧缓冲区)和 color buffer(颜色缓冲区)。

### 2.4.1 GPU 内存架构
寄存器和内存有什么区别?
从物理结构而言，寄存器是 cpu 或 gpu 内部的存储单元，即寄存器是嵌入在 cpu 或者 gpu 中的，而内存则可以独立存在;从功能上而言，寄存器是有限存储 容量的高速存储部件，用来暂存指令、数据和位址。Shader 编成是基于计算机图 形硬件的，这其中就包括 GPU 上的寄存器类型，glsl 和 hlsl 的着色虚拟机版本 就是基于 GPU 的寄存器和指令集而区分的。
![图6](http://img.blog.csdn.net/20170717132722868?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkRhbGFzamE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2.4.2 Z Buffer 与 Z 值
Z buffer 应该是大家最为熟悉的缓冲区类型，又称为 depth buffer，即深度缓 冲区，其中存放的是视点到每个像素所对应的空间点的距离衡量，称之为 Z 值 或者深度值。可见物体的 Z 值范围位于【0，1】区间，默认情况下，最接近眼 睛的顶点(近裁减面上)其 Z 值为 0.0，离眼睛最远的顶点(远裁减面上)其 Z 值为 1.0。使用 z buffer 可以用来判断空间点的遮挡关系，著名的深度缓冲区算 法(depth-buffer method，又称 Z 缓冲区算法)就是对投影平面上每个像素所对 应的 Z 值进行比较的。

Z 值并非真正的笛卡儿空间坐标系中的欧几里德距离(Euclidean distance)， 而是一种“顶点到视点距离”的相对度量。所谓相对度量，即这个值保留了与其他 同类型值的相对大小关系。在 steve Baker 撰写的文章“Learning to love your Z-buffer”中将 GPU 对 Z 值的计算公式描述为:
![公式2](http://img.blog.csdn.net/20170717132834715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQkRhbGFzamE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中 f 表示视点到远裁减面的空间距离， n 表示视点到近裁减面的空间距 离， z 表示视点到顶点的空间距离，N 表示 Z 值精度。

大多数人所忽略的是，z buffer 中存放的 z 值不一定是线性变化的。在正投 影中同一图元相邻像素的 Z 值是线性关系的，但在透视投影中却不是的。在透 视投影中这种关系是非线性的，而且非线性的程度随着空间点到视点的距离增加 而越发明显。

当 3D 图形处理器将基础图元(点、线、面)渲染到屏幕上时，需要以逐行 扫描的方式进行光栅化。图元顶点位置信息是在应用程序中指定的(顶点模型坐 标)，然后通过一系列的过程变换到屏幕空间，但是图元内部点的屏幕坐标必须 由已知的顶点信息插值而来。例如，当画三角形的一条扫描线时，扫描线上的每 个像素的信息，是对扫描线左右端点处已知信息值进行插值运算得到的，所以内 部点的 Z 值也是插值计算得到的。同一图元相邻像素点是线性关系(像素点是 均匀分布的，所以一定是线性关系)，但对应到空间线段上则存在非线性的情况， 如图 7 所示。所示:线段 AE 是某三角面片的两个顶点，投影到屏幕空间对应到 像素 1 和像素 5;光栅化时，需要对像素 2、3、4 进行属性插值，从视点引射线 到空间线段上的交点分别为 B、C、D。从图中可以看出，点 B、C、D 并不是均 匀分布在空间线段上的，而且如果离视点越远，这种差异就越发突出。即，投影 面上相等的步长，在空间中对应的步长会随着离视点距离的增加而变长。所以如 果对内部像素点的 Z 值进行线性插值，得到的 Z 值并不能反应真实的空间点的 深度关系。Z 值的不准确，会导致物体显示顺序的错乱，例如，在游戏中常会看 到远处的一些面片相互交叠。

为了避免或减轻上述的情况，在设置视点相机远裁减面和近裁减面时，两者 的比值应尽量小于 1000。要想解决这个问题，最简单的方法是通过将近截面远 离眼睛来降低比值，不过这种方法的副作用时可能会将眼前的物体裁减掉。

很多图形硬件使用 16 位的 Z buffer，另外的一些使用 24 位的 Z buffer，还有 一些很好的图形硬件使用 32 位的。如果你拥有 32 位的 Z buffer，则 Z 精度 (Z-precision)对你不是一个问题。但是如果你希望你的程序可以灵活的使用各 种层次的图形硬件，那么你就需要多思考一下。
Z 精度之所以重要，是因为 Z 值决定了物体之间的相互遮挡关系，如果没有 足够的精度，则两个相距很近的物体将会出现随机遮挡的现象，这种现象通常称 为“flimmering”或”Z-fighting”。