---
title: "数学分析-导数与微分"
date: 2026-03-1
lastmod: 2026-03-01
draft: false
description: ""
tags: ["导数","微分"]
categories: ["数学分析"]
keywords: ["导数","微分"]
---


# 导数与微分核心概念总结

---

## 1. 导数的定义

$$f'(x_0) = \lim_{\Delta x \to 0} \frac{f(x_0 + \Delta x) - f(x_0)}{\Delta x}$$

**关键点：**
- 差商用的是**函数的实际值** $f(x_0)$，不是极限值,这一点很重要，意味着如果函数在这一点不连续(如跳跃函数),
e.g 对于函数 f(x)= x ,其中x=1,f(x)=3的函数
$\lim_{\Delta x \to 0} \frac{f(1+\Delta x) - f(1)}{\Delta x} = \lim_{\Delta x \to 0} \frac{(1+\Delta x)-3}{\Delta x} = \lim_{\Delta x \to 0} (1-\frac{2}{\Delta x}) = \infty$
导数不存在。但是如果定义中不使用$f(x_0)$，而是使用极限值，那么导数存在。
- $\Delta x \neq 0$（去心邻域），避免分母为零
- $f'(x_0)$ 是一个普通实数

**导数的双重含义：**
- 分析含义：差商的极限，提取线性系数
- 近似含义：从 $f(x_0)$ 出发，预测周围点的值

不过我个人觉得用微分的概念定义导数更加自然，也就是说我们可以规定f'(x) = dy/dx ，即 A = f'(x)，不过这个意味着我们需要证明导数的唯一性。
并且这个更巧妙的地方在于，导数现实的一个维度的变化和自变量的关系，在多维度处理的时候，需要偏导数的概念。

---

## 2. 微分的定义

若存在常数 $A$（与 $\Delta x$ 无关），使得：

$$\Delta y = A \cdot \Delta x + o(\Delta x)$$

则称 $f$ 在 $x_0$ 处**可微**，记线性主部为：

$$dy = A \cdot \Delta x$$

**$\Delta x$ 和 $\Delta y$ 的定义：**

$$\Delta x \in \mathbb{R},\quad \Delta x \neq 0 \quad \text{（任意给定的实数增量）}$$

$$\Delta y = f(x_0 + \Delta x) - f(x_0) \quad \text{（函数的真实增量）}$$

**$d$ 符号的本质：** 取线性主部，主动丢掉高阶无穷小 $o(\Delta x)$，是一种近似操作。

$$\Delta y = \underbrace{A\cdot\Delta x}_{dy,\ \text{线性近似}} + \underbrace{o(\Delta x)}_{\text{高阶误差，被丢掉}}$$


---

## 3. $\Delta x$ 与 $dx$ 的区别

| 符号 | 本质 | 是否趋向零   |
|------|------|---------|
| $\Delta x$ | 任意实数增量，函数的自变量 | 否，是给定的数 |
| $dx$ | 定义为等于 $\Delta x$ | 否，同上    |
| $\Delta y$ | 真实增量，精确值 | 否       |
| $dy$ | 线性近似，丢掉高阶项 | 否       |

**$dy/dx$ 的真实含义：**
我们主要是区分的是分母的 自变量，和分子的因变量。自变量表示的只是任意实数增量，而分子表示是随变化的近似线性结果，这个与$\Delta y$ 最大的不同就是说我们希望dy可以表示一种近似线性结果。

$$\frac{dy}{dx} = \frac{f'(x_0)\cdot\Delta x}{\Delta x} = f'(x_0)$$
    这里说明一下为什么 $dy =f(x)'dx$
   第一步：微分定义只说存在 A                                                                                                                                                                                                          
   
  $$\Delta y = A \cdot \Delta x + o(\Delta x)$$                                                                                                                                                                                          
                                                                                                                                                                                                                                       
  此时 $A$ 是某个常数，还不知道它是什么。      

   第二步：推导 A 的具体值

  两侧除以 $\Delta x$，取极限：

  $$\lim_{\Delta x \to 0} \frac{\Delta y}{\Delta x} = \lim_{\Delta x \to 0} \left(A + \frac{o(\Delta x)}{\Delta x}\right) = A + 0 = A$$

  但左边正是导数定义，所以：

  $$A = f'(x_0)$$

  第三步：代回得到 dy

  $$dy = A \cdot \Delta x = f'(x_0) \cdot \Delta x = f'(x_0) \cdot dx$$



---

## 4. 可微与可导的等价证明（单变量）

### 可微 → 可导

由可微定义 $\Delta y = A\cdot\Delta x + o(\Delta x)$，两侧除以 $\Delta x$：

$$\frac{\Delta y}{\Delta x} = A + \frac{o(\Delta x)}{\Delta x}$$

取极限：

$$\lim_{\Delta x\to 0}\frac{\Delta y}{\Delta x} = A + 0 = A$$

所以 $f'(x_0) = A$ 存在，可导。

### 可导 → 可微

已知 $f'(x_0)$ 存在，令：

$$\alpha(\Delta x) = \frac{\Delta y}{\Delta x} - f'(x_0), \quad \lim_{\Delta x\to 0}\alpha(\Delta x) = 0$$

则：

$$\Delta y = f'(x_0)\cdot\Delta x + \underbrace{\alpha(\Delta x)\cdot\Delta x}_{o(\Delta x)}$$

验证余项：$\lim_{\Delta x\to 0}\frac{\alpha(\Delta x)\cdot\Delta x}{\Delta x} = \lim_{\Delta x\to 0}\alpha(\Delta x) = 0$ 

**结论：单变量中 可微 $\Leftrightarrow$ 可导。**

---

## 5. 复合函数求导（链式法则）

$$[g(f(x))]' = g'(f(x))\cdot f'(x)$$

**证明思路：**
构造 $du/dx = (du /dv )*(dv/dx)$




## 8. 微分、导数、连续、极限存在的关系

$$\text{可微} \Leftrightarrow \text{可导} \Rightarrow \text{连续} \quad (\text{连续定义内含极限存在})$$

每个箭头反过来不成立。

### 可导 → 连续的证明

$$f(t)-f(x) = \frac{f(t)-f(x)}{t-x}\cdot(t-x) \quad (t\neq x,\ \text{普通代数恒等式})$$

取极限（极限乘法透传，两个极限均存在）：

$$\lim_{t\to x}[f(t)-f(x)] = \lim_{t\to x}\frac{f(t)-f(x)}{t-x}\cdot\lim_{t\to x}(t-x) = f'(x)\cdot 0 = 0$$

$$\Rightarrow \lim_{t\to x}f(t) = f(x) \quad \text{（连续的定义）}$$

**核心：有限数 × 0 = 0。导数存在保证"有限数"这个条件。**

### 连续不推出可导

$f(x) = |x|$ 在 $x=0$：连续，但差商 $\frac{|t|}{t}$ 左极限 $-1$，右极限 $+1$，极限不存在，不可导。

**断掉的位置：** 连续给了分子 $\to 0$，但 $\frac{0}{0}$ 型极限是另一个问题。

### 不可微的几何特征

| 类型 | 几何表现 | 原因 |
|------|---------|------|
| 左右导数不等 | 尖点（如 $\|x\|$） | 无统一线性近似 |
| 导数无穷大 | 竖直切线 | 线性系数爆炸 |
| 极限震荡 | 无切线 | 找不到稳定线性系数 |

### 不连续类型

| 类型 | 极限 | 函数值 |
|------|------|-------|
| 可去间断点 | 存在，等于 $L$ | $f(x_0)\neq L$ |
| 跳跃间断点 | 不存在（左右不等） | — |

**可去间断点仍然不可导：** 差商用的是函数实际值 $f(x_0)$，跑偏的值使差商爆炸。

---

## 9. 核心思想一句话

> 导数 = 找到最佳线性近似的系数。可微 = 函数局部可以被线性函数替代，误差丢进高阶无穷小。
如果不可导或者不可微，意味着无法用线性关系近似表示函数周围点(离散)