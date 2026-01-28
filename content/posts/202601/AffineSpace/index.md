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



## 仿射空间基本定义

### 定义 4.1.1 (仿射空间)

设 $V$ 是域 $\mathbb{K}$ 上的 $n$ 维向量空间，$A$ 是一个非空集合，$A$ 中的元素称为**点**(point)。如果存在满足以下条件的加法映射 
$$+: A \times V \to A, \quad(p, x) \mapsto p + x \in A$$

**(1)** $\forall x, y \in V$ 及 $p \in A$，有 $(p+x)+y = p+(x+y)$

**(2)** $\forall p \in A$，$p + 0 = p$

**(3)** $\forall p, q \in A$，存在唯一的 $x \in V$ 使得 $p + x = q$（我们记为 $x = \overrightarrow{pq}$ 或 $x = q - p$）

则称 $A$ 是一个与 $V$ **相伴的仿射空间** (affine space)。我们定义 $A$ 的维数 
$$\dim(A) = \dim(V) = n$$

理解：
从定义理解，A(仿射空间) 通过向量空间V定义 ，但是需要额外结构
1. 向量空间V，拥有完整代数结构，点集A，抽象集合无结构，但是通过作用映射$$+: A \times V \to A $$拥有了结构
2. 定义1确保了点的叠加是结合的，定理二保证零向量不改变点，定理三比较关键，保证了任意两点的向量连接是唯一的
3. 仿射空间忽略了原点，保留了平移和点之间距离的概念
