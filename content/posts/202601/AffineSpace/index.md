---
title: "仿射空间数学理解及证明"
date: 2026-01-28
lastmod: 2026-01-28
draft: false
description: "仿射空间数学理解及证明"
tags: ["affine"]
categories: ["math", "liner algebra"]
keywords: ["math", "liner algebra", "affine Map"]
---





# 仿射空间的数学定义与坐标变换公式详解

**理解模块是笔者个人心得体会，如果有误欢迎留言指出**


## 仿射空间基本定义

### 定义 1 (仿射空间)

设 $V$ 是域 $\mathbb{K}$ 上的 $n$ 维向量空间，$A$ 是一个非空集合，$A$ 中的元素称为**点**(point)。如果存在满足以下条件的加法映射 
$$+: A \times V \to A, \quad(p, x) \mapsto p + x \in A$$

**(1)** $\forall x, y \in V$ 及 $p \in A$，有 $(p+x)+y = p+(x+y)$

**(2)** $\forall p \in A$，$p + 0 = p$

**(3)** $\forall p, q \in A$，存在唯一的 $x \in V$ 使得 $p + x = q$（我们记为 $x = \overrightarrow{pq}$ 或 $x = q - p$）

则称 $A$ 是一个与 $V$ **相伴的仿射空间** (affine space)。我们定义 $A$ 的维数 
$$\dim(A) = \dim(V) = n$$

理解要点：
从定义理解，A(仿射空间) 通过向量空间V定义 ，但是需要额外结构
1. 向量空间V，拥有完整代数结构，点集A，抽象集合无结构，但是通过作用映射$$+: A \times V \to A $$拥有了结构
2. 定义1确保了点的叠加是结合的，定理二保证零向量不改变点，定理三比较关键，保证了任意两点的向量连接是唯一的
3. 仿射空间没有特权原点，保留了平移和点之间距离的概念（定义3）
4. 考虑通过定义,$$+: A \times V \to A, \quad(p, x)$$ 我们可以去理解仿射空间A唯一确定向量空间V，而向量空间本身可以确定无数仿射空间
5. 考虑仿射空间的定义，在向量空间无法相加的点的概念，通过结构化定义, $(p+x)+y = p+(x+y)$  ,其中 $$ x,y \in V ,p \in A $$ 巧妙的结合了点和向量之间的运算关系

### 定义 2 仿射坐标系

设 $V$ 是域 $\mathbb{K}$ 上的 $n$ 维向量空间，$\mathbf{e}_1, \mathbf{e}_2, \ldots, \mathbf{e}_n$ 是 $V$ 的一组基。令 $\mathbb{A}$ 是与 $V$ 相伴的仿射空间，取定点 $o \in \mathbb{A}$，则我们称 $(o, \mathbf{e}_1, \ldots, \mathbf{e}_n)$ 构成了 $\mathbb{A}$ 的一个**仿射坐标系**（或称**仿射标架**，affine coordinate frame），其中 $o$ 称为仿射坐标系的**原点**。


理解：仿射坐标系的原点是任意属于仿射空间集合中的元素，唯一性的证明，即任意点p可以被唯一表示可以通过定义$\forall p, q \in A$，存在唯一的 $x \in V$ 使得 $p + x = q$ ,将p设置为仿射坐标系原点,点p可以被唯一表示(因为$x$是唯一的)，$$ p = o + \sum_{i=1}^{n} x_i \mathbf{e}_i $$我们称 $(x_1, \ldots, x_n)^t$ 是 $p$ 在此仿射坐标系下的**坐标**。


## 核心定理：坐标变换公式

设 $\mathbb{A}$ 是域 $\mathbb{K}$ 上与 $V$ 相伴的 $n$ 维仿射空间，$(o, \mathbf{e}_1, \ldots, \mathbf{e}_n)$ 和 $(o', \mathbf{e}'_1, \ldots, \mathbf{e}'_n)$ 是 $\mathbb{A}$ 的两个仿射坐标系。 设 $p \in \mathbb{A}$ 在 $(o, \mathbf{e}_1, \ldots, \mathbf{e}_n)$ 下的坐标是 $(x_1, \ldots, x_n)^t$，在 $(o', \mathbf{e}'_1, \ldots, \mathbf{e}'_n)$ 下的坐标是 $(x'_1, \ldots, x'_n)^t$，点 $o'$ 在 $(o, \mathbf{e}_1, \ldots, \mathbf{e}_n)$ 下的坐标是 $(b_1, \ldots, b_n)^t$，而且 $(\mathbf{e}'_1, \ldots, \mathbf{e}'_n) = (\mathbf{e}_1, \ldots, \mathbf{e}_n) \cdot A$，则 $$ \begin{pmatrix} x_1 \\ \vdots \\ x_n \end{pmatrix} = A \cdot \begin{pmatrix} x'_1 \\ \vdots \\ x'_n \end{pmatrix} + \begin{pmatrix} b_1 \\ \vdots \\ b_n \end{pmatrix} $$
证明: 
证明分为四步，分别是
1. 利用仿射空间定义 4.1.1 的 (1) 和 (3)，我们有以下三个基本等式：

$$
\begin{aligned}
O + \overrightarrow{OP} &= P \\
O + \overrightarrow{OO'} &= O' \\
O' + \overrightarrow{O'P} &= P
\end{aligned}
$$

2. 现在我们将上述向量关系用坐标表示

$$
\overrightarrow{OP} = (e_1,\ldots,e_n) \begin{pmatrix} x_1\\ x_2\\ \vdots\\ x_n \end{pmatrix}
$$

$$
\overrightarrow{OO'} = (e_1,\ldots,e_n) \begin{pmatrix} b_1\\ b_2\\ \vdots\\ b_n \end{pmatrix}
$$

$$
\overrightarrow{O'P} = (e'_1,\ldots,e'_n) \begin{pmatrix} x'_1\\ x'_2\\ \vdots\\ x'_n \end{pmatrix} = (e_1,\ldots,e_n) A \begin{pmatrix} x'_1\\ x'_2\\ \vdots\\ x'_n \end{pmatrix}
$$

代入提取公共基并由于 $(e_1,...,e_n)$ 是向量空间V的一组基，坐标表示唯一，所以存在唯一解，故证明等式


理解 ： 这个公式深刻的说明了仿射空间里面的线性变化，即在同一个向量空间内，坐标系的变化可以表示为一次线性变化（旋转，缩放，剪切，反射等）加一次平移(向量b)。特别的如果新旧坐标系都是标准正交基$$A^T  A =I , det(A)=1$$的情况下，仿射坐标变换 = 一次刚体旋转+一次平移 ，这个公式也为后面的GIS领域矢量珊格化提供理论保障。齐次坐标的引用也是基于这个公式。（在下一篇文章中我将详细解释在图像处理领域和图形学齐次坐标的用法)
 







