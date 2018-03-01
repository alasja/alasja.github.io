---
title: "四元数"
sidebar: rtr
layout: book
permalink: rtr_ch4_quaternions.html
summary: 第四章.
---

### 4.3 四元数
早在1843年，`William Rowan Hamilton`就发现了作为复数的扩展形式的四元数，直到1985年才被`Shoemake`引入计算机图形领域。四元数使用表达变换的强力形式，在有些情况下，比欧拉角和矩阵的表现都要好，特别是在旋转和方向方面。给定轴和角度，无论是转换成四元素，还是从四元素转换回来都是容易，而欧拉角的两种转换都比较复杂一点。四元数可以对方向进行稳定的插值，这是欧拉角基本没法做到的。

复数包含一个实部和一个虚部。每个复数使用两个实数来表示，第二实数会乘以根号-1。简单来说四元数有四个部分。前三个值跟旋转轴相关，旋转的角度会作用在所有四个分部上面(详见4.3.2)。每个四元素使用四个实数来表示，每个实数表示一个不同的部分。因为四元数有四个元素，我们可以用向量来表示它，为了区分它和向量，它的符号上有个尖角，如$\hat q$。让我们从四元数的数学背景知识讲起，这是后面有趣加有用的变换的基础。

### 4.3.1 数学背景
我们从四元数的定义开始。

定义：四元数$\hat q$可以由以下方式来定义，每种方式都是等价的：

$$
\hat q = (q_v,q_w) = iq_x + jq_y + kq_z + q_w = q_v + q_w,\\
q_v = iq_x + jq_y + kq_z = (q_x, q_y, q_z),\\
i^2 = j^2 = k^2 = -1, \; jk = -kj = i, \; ki = -ik = j, \; ij = -ji = k. \tag{4.28}
$$

<font color="tomato">变量$q_w是\hat q$的实部。$q_v$是虚部，i, j, k被称为虚部单位。</font>

对于虚部的$q_v$，我们可以把它当做是普通的向量一样进行操作，例如向量相加，缩放，点乘，叉乘等等。根据四元数的定义，两个四元数相乘的计算步骤如下4.29所示。要注意虚部单位的相乘是不遵从交换律的。

$$
乘法：\\
\hat q \hat r = (q_v, q_w)(r_v, r_w) \\
= (q_v \times r_v + r_wq_v + q_wr_v, q_wr_w - q_v \cdot r_v)\\
= i(q_yr_z - q_zr_y + r_wq_x + q_wr_x)\\
+j(q_zr_x - q_xr_z + r_wq_y + q_wr_y)\\
+k(q_xr_y - q_yr_x + r_wq_z + q_wr_z)\\
+ q_wr_w - q_xr_x - q_yr_y - q_zr_z.  \tag{4.29}
$$

从上面的等式可以看到，我们同时使用了点乘和叉乘来完成两个四元数的乘法计算。四元数的相加、共轭、规范化和单位定义如下：

$$
Addition:\quad \hat q + \hat r = (q_v, q_w) + (r_v, r_w) = (q_v + r_v, q_w + r_w). \\
Conjugate:\quad \hat q^* = (q_v, q_w)^* = (-q_v, q_w).\\
Norm:\quad n(\hat q) = \sqrt{\hat q \hat q^*} = \sqrt{\hat q^* \hat q} = \sqrt{q_v \cdot q_v + q_w^2} = \sqrt{q_x^2 + q_y^2 + q_z^2 + q_w^2}.\\
Identity:\quad \hat i = (0,1). \tag{4.30}
$$

规范化$n(\hat q) = \sqrt{\hat q \hat q^*}$的结果很简洁（如上），四元数的虚部没有了，只剩下一个实部。规范化有时也记作$n(\hat q) = \Vert\hat q\Vert$[808]。从上面的定义我们能推导出四元数的逆，记作$\hat q^{-1}$。逆必然满足等式$\hat q^{-1}\hat q = \hat q \hat q^{-1} = 1$，这是逆的乘法属性。

$$ 
n(\hat q)^2 = \hat q \hat q^* \\
\frac{\hat q \hat q^*}{n(\hat q)^2} = 1. \tag{4.31}
$$

所以我们得出四元数的逆的表达式：

$$
Inverse:\qquad \hat q^{-1} = \frac{1}{n(\hat q)^2}\hat q^*. \tag{4.32}
$$

逆的计算公式使用标量的计算($\frac{1}{n(\hat q)^2}$这部分是个标量数值)，这可以通过4.29中等等式来推导得出：$s\hat q = (0, s)(q_v, q_w) = (sq_v, sq_w)，且\hat q s= (q_v, q_w)(0, s) = (sq_v, sq_w)$，这表示标量跟四元数的乘法是遵守交换律的：$s\hat q = \hat q s = (sq_v, sq_w)$。

下面这些推论很容易从定义中推导得出：

**共轭推论：**
$$
(\hat q^*)^* = \hat q,\\
(\hat q + \hat r)^* = \hat q^* + \hat r^*,\\
(\hat q\hat r)^* = \hat r^* \hat q^*. \tag{4.33}
$$
**规范化推论：**
$$
n(\hat q^*) = n(\hat q),\\
n(\hat q\hat r) = n(\hat q)n(\hat r). \tag{4.34}
$$
**乘法定律：**
$$
\hat p(s\hat q + t\hat r) = s\hat p \hat q + t\hat p \hat r,\\
(s\hat p + r\hat q)\hat r = s\hat q \hat r + t \hat q \hat r. \tag{4.35}
$$

**结合律：**

$$
\hat p(\hat q \hat r) = (\hat p \hat q)\hat r.
$$

一个单位四元数，就是$n(\hat q) = 1$。那么这个四元数可以写成如下形式：

$$
\hat q = (sin\theta u_q, cos\theta) = sin\theta u_q + cos\theta.   \tag{4.36}
$$

对于三维向量$u_q$来说，它的模$\Vert u_q \Vert = 1$，因为：

$$
n(\hat q) = n(sin\theta u_q, cos\theta) = \sqrt{sin^2\theta(u_q \cdot u_q) + cos^2\theta} \\
= \sqrt{sin^2\theta + cos^2\theta} = 1.  \tag{4.37}
$$

当且仅当$u_q \cdot u_q = 1 = \Vert u_q \Vert^2$的时候上述等式才成立。在下一节中我们会看到，单位四元数在构建旋转和朝向的是多么的高效和完美。在这之前，我们再介绍一些单位四元数的操作。

对于复数，二位的单位向量可以写作：$cos\theta + i sin\theta = e^{i\theta}$。所以可以有下列等式：

$$
\hat q = sin\theta u_q + cos\theta = e^{\theta u_q}. \tag{4.38}
$$

根据4.38，下面列出的单位四元数的对数和指数操作：

**对数：**$\qquad log(\hat q) = log(e^{\theta u_q}) = \theta u_q.$

**指数：**$
\qquad \hat q^t = (sin\theta u_q + cos\theta)^t = e^{\theta t u_q} = sin(\theta t)u_q + cos(\theta t). \tag {4.39}
$

### 4.3.2 四元数变换
我们接下来要学习四元数的一个子集，单位长度的四元数，也就是单位四元数。单位四元数最重要的特点是可以用来表达任意的三维旋转，而且这个表达非常的简洁又简单。

现在我们来看看为什么单位四元数在旋转和方向上这么好用。首先，把有四个分量的点或向量$p = (p_x \ p_y \ p_z \ p_w)^T$放入到四元数$\hat p$中，我们再假设我们已经有一个单位四元数$\hat q = (sin\theta u_q, cos\theta)$。于是表达式4.40：

$$
\hat q \hat p \hat q^{-1} \tag{4.40}
$$

表示把$\hat p$(也就是点或向量p)绕着轴$u_q$旋转了角度$2\theta$。注意，因为$\hat q$是单位四元数，所以有$\hat q^{-1} = \hat q^*$。这个旋转显然可以表示绕任意轴旋转，见图4.8

![图](/images/RTR3.04.08.png)
图4.8

任何非0系数的$\q$都会产生相同的变换，意思是$\hat q 跟 -\hat q$表示相同的旋转。反向的轴，和反向的$q_w$，构建的四元素实际表示的旋转跟原来的完全一致。这也表面从矩阵中导出一个四元数的话，得到的结果可能是$\hat q 或 -\hat q$

给定两个单位四元数$\hat q 和 \hat r$，先作用q再作用r到p上的表达式如下：

$$
\hat r(\hat q \hat q \hat q^*) \hat r^* = (\hat r \hat q)\hat p(\hat r \hat q)^* = \hat c \hat p \hat c^* \tag{4.41}
$$

这里，$\hat c = \hat r \hat q$表示单位四元数q和r连接之后的单位四元数。



### 4.3.2.1 矩阵转换

基于一些硬件层面有实现矩阵相乘，以及矩阵相乘比等式4.40要更加高效，所以我们需要一些转换方法来把四元数转换成矩阵，或是反向变换。一个四元数$\hat q$，可以使用等式4.42来把它变换成矩阵M：

$$
M^q = \begin{pmatrix}
1-s(q_y^2 + q_z^2) & s(q_xq_y - q_wq_z) & s(q_xq_z + q_wq_y) \\
s(q_xq_y + q_wq_z) & 1-s(q_x^2 + q_z^2) & s(q_yq_z - q_wq_x) \\
s(q_xq_z - q_wq_y) & s(q_yq_z + q_wq_x) & 1-s(q_x^2 + q_y^2) \\
0 & 0 & 0 & 1 \end{pmatrix}. \tag{4.42}
$$

这里，缩放系数s为$s = 2/n(\hat q)$。对于单位四元数，上面表达式变为：

$$
M^q = \begin{pmatrix}
1-2(q_y^2 + q_z^2) & 2(q_xq_y - q_wq_z) & 2(q_xq_z + q_wq_y) \\
2(q_xq_y + q_wq_z) & 1-2(q_x^2 + q_z^2) & 2(q_yq_z - q_wq_x) \\
2(q_xq_z - q_wq_y) & 2(q_yq_z + q_wq_x) & 1-2(q_x^2 + q_y^2) \\
0 & 0 & 0 & 1 \end{pmatrix}. \tag{4.43}
$$

当四元数构造之后，就不用再计算任何的三角函数了，所以这个转换过程是相当高效的。

反向的转换，从矩阵M变换成单位四元数，需要计算的就稍微多一点点了。计算的关键部分是从4.43中的矩阵得出的下列三个等式：

$$
m_{21}^q - m_{12}^q = 4(q_wq_x),\\
m_{02}^q - m_{20}^q = 4(q_wq_y),\\
m_{10}^q - m_{01}^q = 4(q_wq_z).  \tag{4.44}
$$

要解这些等式的关键是$q_w$是已知的，然后向量$v_q$就能被计算出来，四元数也就推算出来。$M^q$的trace的计算如下：

$$
tr(M^q) = 4 - 2s(q_x^2 + q_y^2 + q_z^2) = 4(1 - \frac{q_x^2 + q_y^2 + q_z^2}{q_x^2 + q_y^2 + q_z^2 + q_w^2})\\
= \frac{4q_w^2}{q_x^2 + q_y^2 + q_z^2 + q_w^2} = \frac{4q_w^2}{n(\hat q)}. \tag{4.45}
$$

所以单位四元数的各个分量的表达式如下：

$$
q_w = \frac{1}{2}\sqrt{tr(M^q)}, \qquad q_x = \frac{m_{21}^q - m_{12}^q}{4q_w},\\
q_y = \frac{m_{02}^q - m_{20}^q}{4q_w}, \qquad q_z = \frac{m_{10}^q - m_{01}^q}{4q_w}.
 \tag{4.46}
$$

为了得到稳定的数值，要尽量避免除以非常小的数字。所以，我们先设$t = q_w^2 - q_x^2 - q_y^2 - q_z^2$，那么可得：

$$
m_{00} = t + 2q_x^2,\\
m_{11} = t + 2q_y^2,\\
m_{22} = t + 2q_z^2,\\
u = m_{00} + m_{11} + m_{22} = t + 2q_w^2.   \tag{4.47}
$$

这表示$m_{00},m_{11},m_{22}，u$中最大的一个，也代表了$q_x,q_y,q_z,q_w$中最大的一个。如果$q_w$是最大的，则可以使用表达式4.46来计算除法得到四元数。其他情况下，我们先看下面的等式：

$$
4q_x^2 = +m_{00} - m_{11} - m_{22} + m_{33},\\
4q_y^2 = -m_{00} + m_{11} - m_{22} + m_{33},\\
4q_z^2 = -m_{00} - m_{11} - m_{22} + m_{33},\\
4q_w^2 = tr(M_q).  \tag{4.48}
$$

我们用这几个等式来计算出$q_x,q_y,q_z$中最大的一个，然后使用表达式4.44来计算四元数结果。噢，很幸运，已经有代码实现这个功能了——详见本章的`扩展阅读和资源`。

### 4.3.2.2 球面线性插值
球面线性插值就是，给定两个单位四元数$\hat q,\hat r$以一个参数$t \in [0, 1]$，然后通过插值结算出一个四元数结果。这对动画对象非常有用。但是这在相机的朝向上就不是那么好用了，因为插值过程中可能会改变相机的`up`向量，而产生一种倾斜，令人不舒服的效果。

这个结果$\hat s$可以用以下表达式表示：

$$
\hat s(\hat q, \hat r, t) = (\hat r \hat q^{-1})^t \hat q.  \tag{4.49}
$$

不过，在软件中的实现一般使用下面这种更合适的形式，`slerp`表示球面线性插值：

$$ 
\hat s(\hat q, \hat r, t) = slerp(\hat q, \hat r, t) = \frac{sin(\theta(1-t))}{sin\theta}\hat q + \frac{sin(\theta t)}{sin\theta} \hat r.  \tag{4.50}
$$

为了计算等式中的$\theta$，可以使用这个等式：$cos \theta = q_xr_x + q_yr_y + q_zr_z + q_wr_w$。因为$t \in [0,1]$，`slerp`函数计算出的这些四元数，在四维单位球上构成了$\hat q(t=0)到\hat r(t=1$之间最小圆弧的部分(也就是插值的结果都在夹角范围内)。这个弧线位于，由$\hat q和\hat r$及原点构成的面与四维单位球体所相交的圆上。图4.9展示了这一点。计算出的旋转四元数绕以固定速度绕特定轴旋转，这种类型的匀速曲线，就做`测地线弧度(geodesic curve)`。

![图](/images/RTR3.04.09.png)

（<font color="DarkGoldenRod">图4.9. 单位四元数在这个四维球上表现为一个点。slerp函数对四元数进行插值，插值的结果在球上组成了一个大圆弧。注意对$\hat q_1 到\hat q_2$插值，与对$\hat q_1 到 \hat q_3 到\hat q_2$插值，是完全不一样的，即是最终结果是指向同一个方向。</font>）

`slerp`函数可以完美的对两个四元数进行插值，并保持良好的特效（固定轴，固定速度）。这种情况不同于对多个欧拉角进行插值。实践中，通过三角函数来直接计算`slerp`是非常耗时的。Li[771,772]提供了一种非常迅速的方法来计算`slerp`，并且不牺牲任何精度。

当超过两个四元数的时候，比方$\hat q_0, \hat q_1, ..., \hat q_{n-1}$，我们想要从$\hat q_0到\hat q_1再到\hat q_2直到\hat q_{n-1}$来进行插值，slerp函数使用一种简单的插值方式。当我们处理某个四元数的时候，例如$\hat q_i$，我们可以使用$\hat q_{i-1}和\hat q_i$作为参数来插值。处理完$\hat q_i$，则使用$\hat q_i和\hat q_{i+1}$作为参数来插值。这样的插值在节点变换的时候会引起抖动，可以再图4.9中看到。这跟点的线性插值很相似，在13章中将会介绍到，我们可以先看一下插图13.2。下面一些段落，在你看完13章的样条线之后可能会想回来再看看。

![图](/images/RTR3.13.02.png)
图13.2

一个更好的插值方式是使用曲线类型的插值。我们在$\hat q_i 和 \hat q_i+1之间引入\hat a_i 和\hat a_i{+1}$。球面插值可以这些四元数来定义。幸运的是，这些新引入的四元数可以通过下面的等式来计算：

$$
\hat a_i = \hat q_i exp \left[ - \frac{log(\hat q_i^{-1} \hat q_{i-1}) + log(\hat q_i^{-1} \hat q_{i+1})}{4} \right]. \tag{4.51}
$$

这里的$\hat q_i和\hat a_i$将被用来使用一种平滑的弧线来做球面插值。如下表达式所示：

$$
squad(\hat q_i, \hat q_{i+1}, \hat a_i, \hat a_{i+1}, t) = \\
slerp(slerp(\hat q_i, \hat q_{i+1}, t), slerp(\hat a_i, \hat a_{i+1}, t), 2t(1-t)). \tag{4.52}
$$

如上所示，`squad`函数是通过重复使用`slerp`球面插值来构造的（13.1.1介绍了对点进行重复的线性插值）。整个插值过程会依次处理初始的四元数$\hat q_i, i \in [0,...,n-1]$，但是不包括$\hat a_i$，这些四元数的引入只是为了表明初始四元数的切线四元数。

### 4.3.2.3 向量旋转变换
一个很常用的操作是从一个方向s 使用最短的路径变换到另一个方向t。四元数的数学特性极大的简化这个问题，这也表明了四元数和这种表现的紧密关系。首先我们把s和t规范化。然后计算单位旋转轴u，计算方式为$u = (s \times t) / \Vert s \times t \Vert.$，然后根据$e = s \cdot t = cos(2\theta), \Vert s \times t \Vert = sin(2\theta)$来计算，其中$2\theta$就是s和t的夹角。旋转s到t的四元数为$\hat q = (sin(\theta) u, cos(\theta))$。也就是$\hat q = (\frac{sin(\theta)}{sin(2\theta)}(s \ times t), cos(\theta))$，我们使用半角关系以及附录B.9中三角函数相关定义简化之后得到：

$$
\hat q = (q_v, q_w) = \left( \frac{1}{\sqrt{2(1+e)}}(s \times t), \frac{\sqrt(2(1+e))}{2}  \right). \tag{4.53}
$$

直接通过这种方式来构造四元数（与规范化s与t的叉乘相比），避免了当s与t基本指向同一方向的时候引起的数值稳定问题。s与t指向相反方向时，两种方法都会有数值稳定问题，因为可能会导致除以0。当发现这种特殊情况时，任何垂直于s的旋转轴也可以用于t。

有些情况下，我们需要旋转s到t的矩阵表现形式。等式4.43经过一些代数和三角函数计算简化后，我们得到以下矩阵：

$$
R(s,t) = \begin{pmatrix}
e + h v_x^2 & hv_xv_y - v_z & hv_xv_z + v_y & 0 \\
hv_xv_y + v_z & e + h v_y^2 & hv_yv_z - v_x & 0 \\
hv_xv_z - v_y & hv_yv_z + v_x & e + h v_x^2 & 0 \\
0 & 0 & 0 & 1 
\end{pmatrix}  \tag{4.54}
$$

在这个等式中，我们使用下面这些计算：

$$
v = s \times t,\\
e = cos(2\theta) = s \cdot t, \\
h = \frac{1-cos(2\theta)}{sin^2(2\theta)} = \frac{1 - e}{v \cdot v} = \frac{1}{1 + e}.  \tag{4.55}
$$

如上所示，所有的平方根和三角函数都在简化中消除了，所以这是构建矩阵的一个相当有效率的方法。

记住，我们必须要注意s和t平行或者近乎平行的情况，这种情况下$\Vert s \times t \Vert \approx 0。如果\theta \approx 0$，那我们直接使用单位矩阵。但是如果$2\theta \approx \pi$，那我们可以绕任意轴旋转180度。这个轴可以通过s与其他任何不与s平行的向量叉乘得到（见4.2.4）。Moller和Hughes介绍了`Householder`矩阵来处理这种特殊情况的另一种方法。


**举例：设置相机的位置和朝向**。假设一个虚拟相机（或视点）的初始位置为$(0 \; 0 \; 0)^T$，初始朝向v朝着z轴负方向，记$v = (0 \; 0\; -1)^T$。现在，我么需要把相机移到一个新位置p，看向一个新方向w。先确定相机的朝向，我们可以把初始朝向旋转到目标朝向。使用$R(v, w)$来完成旋转。位置移动可以简单平移到p点，所以整个操作矩阵为$X = T(p)R(v,w)$。实践中，在第一个旋转之后，我们可能需要另一个向量到向量的旋转来操作相机的up向量，以便保持up向量朝上，或其他需要的方向。


