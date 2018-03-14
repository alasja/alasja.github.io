---
title: "5.5 着色器"
sidebar: rtr
layout: book
permalink: rtr_ch5_shading.html
---

### 5.5 着色器
着色是使用一个公式，依据材质属性和光源，来计算视线v方向上的输出辐射度$L_0$。第七章会探讨很多可行的着色公式。这里我们使用一个相对简单的例子来作为说明，并说明如何使用在可编程着色器中实现它。

我们使用的公式中包括漫反射和高光部分。漫反射部分很简单。根据上一节中的定义，我们可以得到漫反射出射度$M_{diff}$是光线辐照度$E_L$，它的方向*l*和漫反射颜色的函数：

$$
M_{diff} = c_{diff} \bigotimes E_L \overline{cos} \theta_i.    \tag{5.3}
$$

这里$\bigotimes$表示对向量按位乘（这里都是RGB向量）。

因为我们假定漫反射在各个方向上的辐射度$L_{diff}$都是一样，于是得到以下关系式（7.5.1会详细解释原因）：

$$
L_{diff} = \frac{M_{diff}}{\pi}. \tag{5.4}
$$

将公式5.3，5.4连起来，我们可以得到：

$$
L_{diff} = \frac{c_{diff}}{\pi} \bigotimes E_L \overline{cos} \theta_i. \tag{5.5}
$$

这种漫反射着色方式也被称为`Lambertian`， 它遵从`兰伯特定律(Lambert's law)`，这个定律定义了理想漫反射表面输出的辐射率跟$\overline{cos}\theta_i$成正比。注意，这个限制范围的余弦因子（一般使用法线*n*与光线*l*点乘得到，使用max(n·l,0）并不是兰伯特表面特有的；正如我们看到的，它一般使用在辐照度上。兰伯特表面的特点是输出的辐射率正比与辐照度。只要你看过兰伯特的着色代码，就会发现除了$1/\pi$跟等式5.5很相像。这个因子在实时渲染中一般已经包含在$E_L$中了。（7.5节会深入相关细节）

类似公式5.3，高光的等式如下：

$$
M_{spec} = c_{spec} \bigotimes E_L \overline{cos} \theta_i.    \tag{5.6}
$$

高光项比漫反射要复杂一些，因为它对方向的依赖。在这里，我们使用`半程向量(half vector)h`，之所以这么叫是因为它是视线向量*v*和光线向量*l*的中值。计算方法如下：

$$
h = \frac{l + v}{\Vert l + v \Vert}.        \tag{5.7}
$$

使用*l+v*除以他们的长度，得到单位向量。

下面的等式用来定义高光辐射度$L_{spec}$的分布情况（7.6节会解释为什么会是这样）：

$$
L_{spec}(v) = \frac{m + 8}{8 \pi} \overline{cos}^m \theta_h M_{spec}.   \tag{5.8}
$$

其中$\theta_h$是*h* 和 *n* 之间的夹角，见图5.13 。

![图](/images/RTR3.05.13.png)
图5.13

需要注意的是，与$L_{diff}不同，L_{spec}$取决于视线向量*v*（非直接的，实际是取决于向量*h*)。$L_{spec}$随着向量*h，v*的夹角减小而增大。变化的速度则由参数*m*决定，它代表表面的光泽度。增大*m*会使得高光更小、更亮。结合等式5.6和5.8得到高光项的着色计算公式：

$$
L_{spec}(v) = \frac{m + 8}{8 \pi} \overline{cos}^m \theta_h c_{spec} \bigotimes E_L \overline{cos} \theta_i.   \tag{5.9}
$$

由上面可得，完整的计算这两项的着色公式是：

$$
L_o(v) = \left( \frac{c_{diff}}{\pi} + \frac{m + 8}{8 \pi} \overline{cos}^m \theta_h c_{spec} \right) \bigotimes E_L \overline{cos} \theta_i.   \tag{5.10}
$$

这个着色等式跟`Blinn-Phong`着色很相似，它是由Blinn在1977年的文章中首次提出：
（<font color="DarkGoldenRod">注：Blinn的论文实际上说的是一个完全不同着色公式，`Blinn-Phong`等式只是在提及之前工作（Phong前两年发表的多个着色公式）的部分，简略的提了一下。但是神奇的是，这个`Blinn-Phong`公式迅速的被大家所接受，以至于他的论文的主要内容都被忽略了，直到四年后`Cook-Torrance`论文才重见天日 </font>）

$$
L_o(v) = \left( \overline{cos} \theta_i c_{diff} + \overline{cos}^m \theta_h c_{spec} \right) \bigotimes B_L.  \tag{5.11}
$$

这里使用$B_L$而不是$E_L$由一些历史原因，这个公式不常用在物理光照中，它会有一种把光源增亮的效果。不过这个公式出现的地方没有处理任何关于光强度的问题。如果我们令$B_L = E_L/\pi$，则它跟5.10就更加相像了，只有两个不同：少了$(m + 8)/8$项，以及高光项没有乘上$\overline{cos}\theta_i$。第七章会更详细的介绍这些不同之处。


### 5.5.1 着色公式实现
5.10这类等式一般在顶点着色器或像素着色器中求值。本章会实现这个等式的一个简化版本，即整个网格的材质属性($c_{diff}, c_{spec}和m$)都是固定的。第六章会讨论一些特别的材质属性使用方式。

等式5.10只计算了单个光源。不过，场景经常会包含多个光源。光线自然而然需要叠加，所以我们可以叠加各个光源的贡献，以得到整体的着色公式：

$$
L_o(v) = \sum_{k=1}^n \left( \left( \frac{c_{diff}}{\pi} + \frac{m + 8}{8 \pi} \overline{cos}^m \theta_{h_k} c_{spec} \right) \bigotimes E_{L_k} \overline{cos} \theta_{i_k} \right). \tag{5.12}
$$

其中$\theta_{h_k}$表示第k个光源的$\theta_h$。

7.9节会讨论多光源的一些问题和相关实现选项。




