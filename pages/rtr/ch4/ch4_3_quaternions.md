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




**矩阵转换**

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