---
title: "数据结构-多维数组的超平面视角：从索引到地址的映射"
date: 2026-02-17
lastmod: 2026-02-17
draft: false
description: "数据结构-多维数组的超平面视角：从索引到地址的映射"
tags: ["数组"]
categories: ["data-struct"]
keywords: ["数据结构", "数组"]
---


# 数据结构-多维数组的超平面视角：从索引到地址的映射


## 核心思想

多维数组在逻辑上是嵌套的子空间结构，在物理上是一段连续的内存。

**寻址的本质：每一维的索引跳过若干个完整的子空间；未满的那个子空间，进入下一维继续定位，直到 0 维。**


## 什么是多维数组？

多维数组是一个定义在离散笛卡尔积上的函数：

$$A: D \to V$$

其中：

$$D = \{0,...,b_1-1\} \times \{0,...,b_2-1\} \times ... \times \{0,...,b_n-1\}$$（定义域）

$V$ = 数据类型的值域（如 int, float）

- **$b_m$** = 第 $m$ 维的大小（该维度拆出的子空间个数）
- **维数** = 笛卡尔积的阶数（坐标分量的个数）
- **元素**由一个 $n$ 元组 $(i_1, i_2, ..., i_n)$ 唯一确定


## 逻辑结构：降维切片

### 几何图解（三维）

```
       k轴 (b₃=4)
       ↑
       │   ┌─────────┐
       │  ╱         ╱│
       │ ╱  平面1  ╱ │
       │┌─────────┐  │        A[2][3][4]
       ││  平面0  │  │
       ││         │  │        总共 2×3×4 = 24 个元素
       ││         │ ╱         连续存放在内存中
       │└─────────┘
       └──────────→ j轴 (b₂=3)
      ╱
     ╱ i轴 (b₁=2)
```

### 降维过程：每次固定一个轴，维度降低一级

对于 `A[b₁][b₂][b₃]`，访问 `A[i][j][k]` 的过程：

**第1步：固定 i（选第几个平面）— 3维 → 2维**

```
  i=0 → 取第0个平面（12个元素）
  i=1 → 取第1个平面（12个元素）

  跳过 i 个完整的 (n-1)维 子空间 [3][4]
  第 i+1 个平面未满 → 进入内部继续定位
```

**第2步：固定 j（选第几行）— 2维 → 1维**

```
  j=0 → 取第0行（4个元素）
  j=1 → 取第1行（4个元素）
  j=2 → 取第2行（4个元素）

  跳过 j 个完整的 (n-2)维 子空间 [4]
  第 j+1 行未满 → 进入内部继续定位
```

**第3步：固定 k（选第几个元素）— 1维 → 0维**

```
  k=0,1,2,3 → 跳过 k 个 (n-3)维 子空间（单个元素）
  → 到达 0 维，定位完成
```

### 数学表述

$$D_3 = \{0,...,b_1-1\} \times \{0,...,b_2-1\} \times \{0,...,b_3-1\} \quad \text{(3维)}$$

$$\downarrow \text{ 固定 } i = i_0$$

$$D_2 = \{0,...,b_2-1\} \times \{0,...,b_3-1\} \quad \text{(2维平面)}$$

$$\downarrow \text{ 固定 } j = j_0$$

$$D_1 = \{0,...,b_3-1\} \quad \text{(1维行)}$$

$$\downarrow \text{ 固定 } k = k_0$$

$$D_0 = \{·\} \quad \text{(0维标量)}$$

> 每次索引 = 固定一个坐标轴 = 超平面切片 = 维度降低一级


## 物理存储：连续线性内存

所有元素在内存中连续存储，没有任何"分界"：

```
内存布局（行优先）：
[a₀₀₀][a₀₀₁][a₀₀₂][a₀₀₃][a₀₁₀][a₀₁₁]...[a₁₀₀][a₁₀₁]...
```

"平面"、"行"只是逻辑标签，物理上是一串连续的字节。


## 地址计算公式

### 核心逻辑：逐层跳过 + 逐层细化

以 `A[2][3][4]` 访问 `A[1][2][3]` 为例。

每一维的索引，跳过若干个完整的子空间；未满的那个子空间，进入下一维继续定位，直到 0 维：

```
第1步：i=1
  跳过 1 个完整的 (n-1)维 子空间 [3][4]
  → 第 2 个平面未满，进入内部继续定位

第2步：j=2
  跳过 2 个完整的 (n-2)维 子空间 [4]
  → 第 3 行未满，进入内部继续定位

第3步：k=3
  跳过 3 个完整的 (n-3)维 子空间（单个元素）
  → 到达 0 维，定位完成
```

偏移量 = 每一步跳过的子空间个数 × 该子空间的大小，逐层累加：

```
偏移 = i × size(n-1维子空间) + j × size(n-2维子空间) + k × size(n-3维子空间)
     = 1 × (3×4)            + 2 × (4)              + 3 × (1)
     = 12 + 8 + 3
     = 23 个元素
```

转成字节：$LOC = LOC_0 + 23 \times L = LOC_0 + 92$

### 子空间大小的递推

每一级子空间的大小 = 下一级的个数 × 下一级的大小：

| 子空间维度 | 大小（元素数） | 递推关系 | 具体值 |
|-----------|---------------|---------|--------|
| 0维（元素） | 1 | 基础单位 | 1 |
| 1维（行） | $b_3 \times 1 = b_3$ | $b_3$ 个元素 | 4 |
| 2维（平面） | $b_2 \times b_3$ | $b_2$ 个行 | 12 |

转成字节就是权重系数 $c_m$（乘以 $L$）：

$$c_3 = 1 \times L = 4, \quad c_2 = b_3 \times c_3 = 16, \quad c_1 = b_2 \times c_2 = 48$$

### 三维公式

$$LOC = LOC_0 + i \times c_1 + j \times c_2 + k \times c_3$$

每一项的含义：**该维索引（跳过几个子空间） × 该维子空间的字节大小** = 跳过的字节数。

### 推广到 n 维

对于 `A[b₁][b₂]...[bₙ]`，访问 `A[i₁][i₂]...[iₙ]`：

$$LOC = LOC_0 + \sum_{m=1}^{n} c_m \cdot i_m$$

展开权重：

$$LOC = LOC_0 + \sum_{m=1}^{n} \left( \prod_{p=m+1}^{n} b_p \right) \cdot i_m \cdot L$$

> $m$ 是求和的循环变量（从第 1 维遍历到第 $n$ 维），不是数组索引 $j$ 或 $k$。
> $\prod_{p=m+1}^{n} b_p$ 就是第 $m$ 维子空间的大小（元素个数），乘以 $L$ 得到字节数。

逻辑始终不变：**从最外层到最内层，每层跳过 $i_m$ 个完整子空间，未满的进入下一层继续定位。**


## 权重系数 $c_m$（递推形式）

定义 $c_m$ = 第 $m$ 维走一步跳过的**字节数**：

$$c_n = L, \quad c_m = b_{m+1} \times c_{m+1} \quad (m = n-1, n-2, ..., 1)$$

其中 $b_{m+1}$ 是第 $m$ 维的子空间内部有多少个下一级子空间，$c_{m+1}$ 是每个下一级子空间的字节大小。

### $b_m$ 与 $c_m$ 的关系

| 符号    | 含义                        | 类比（书架）       |
| ----- | ------------------------- | ------------ |
| $b_m$ | 第 $m$ 维有**几个**子空间（结构形状）   | 第 $m$ 级有几格   |
| $c_m$ | 第 $m$ 维走一步跳**多少字节**（寻址步长） | 第 $m$ 级一格有多宽 |
| $i_m$ | 第 $m$ 维选**第几个**（具体索引）     | 第 $m$ 级选第几格  |
| $L$   | 递推起点，最底层单个元素的字节大小         | 每本书的厚度       |

### 具体例子：`A[2][3][4]`，`sizeof(int) = 4`

**计算权重（从内往外推）：**

$$c_3 = L = 4 \quad \text{（1个元素 = 4字节）}$$

$$c_2 = b_3 \times c_3 = 4 \times 4 = 16 \quad \text{（1行有 $b_3=4$ 个元素 = 16字节）}$$

$$c_1 = b_2 \times c_2 = 3 \times 16 = 48 \quad \text{（1层有 $b_2=3$ 行 = 48字节）}$$

**计算 `A[1][2][3]` 的地址：**

$$LOC = LOC_0 + 1 \times 48 + 2 \times 16 + 3 \times 4 = LOC_0 + 48 + 32 + 12 = LOC_0 + 92$$

分解理解：
- $1 \times 48$：跳过 1 个完整平面 `[3][4]`（进入第1层）
- $2 \times 16$：平面未满，跳过 2 整行 `[4]`（进入第2行）
- $3 \times 4$：行未满，跳过 3 个元素（到达目标）


## 子空间的计数公式

### k 维子空间的个数

$$N(\text{k维子空间}) = \prod_{i=1}^{n-k} b_i$$

含义：从第 1 维到第 $(n-k)$ 维依次固定，第 $i$ 维沿轴跨越 $b_i$ 个位置，可以固定在 $0$ 到 $b_i-1$ 的任意一个位置，所有组合的总数就是 $\prod_{i=1}^{n-k} b_i$。

### 每个 k 维子空间的大小

$$Size(\text{k维子空间}) = \prod_{i=n-k+1}^{n} b_i$$

含义：固定前 $(n-k)$ 维后，剩下的后 $k$ 维自由变化，元素总数 = 后 $k$ 个维度大小之积。下标从 $n-k+1$ 开始是因为取的是**后 $k$ 个维度**。

### 验证关系

$$N_k \times Size_k = \prod_{i=1}^{n} b_i = \text{总元素数} \quad \checkmark$$

### 示例：`A[2][3][4]`

| 维度 | 子空间个数 | 每个大小 | 总元素数 |
|------|-----------|---------|---------|
| 3维（整体） | 1 | $2 \times 3 \times 4 = 24$ | 24 |
| 2维（平面） | $b_1 = 2$ | $b_2 \times b_3 = 3 \times 4 = 12$ | 24 |
| 1维（行） | $b_1 \times b_2 = 2 \times 3 = 6$ | $b_3 = 4$ | 24 |
| 0维（点） | $b_1 \times b_2 \times b_3 = 24$ | 1 | 24 |


## 与混合进制数系统的类比

数组地址计算本质上是**混合进制（Mixed-radix）**的位置表示。

十进制数 253（每位基数都是 10）：

$$253 = 2 \times 10^2 + 5 \times 10^1 + 3 \times 10^0$$

数组偏移 `A[i][j][k]`（每维"基数"不同：$b_1, b_2, b_3$）：

$$offset = i \times (b_2 \times b_3) + j \times b_3 + k \times 1$$

给定偏移量反推索引（类似进制转换）：

$$i = \lfloor offset \;/\; (b_2 \times b_3) \rfloor$$

$$j = \lfloor (offset \bmod (b_2 \times b_3)) \;/\; b_3 \rfloor$$

$$k = offset \bmod b_3$$


## 代码实现
``` cpp
//
// Created by elliot on 2/16/26.
//

#ifndef DATA_STRUCTURE_LEARN_SEQ_ARRAY_HPP
#define DATA_STRUCTURE_LEARN_SEQ_ARRAY_HPP
#include <cstdint>
#include <memory>
#include <stdexcept>
#include <vector>
#include <initializer_list>

// N维数组 - 顺序存储（行优先，一维扁平数组）
template <typename T>
class SeqArray {
public:
    // InitArray - 构造n维数组，bounds为各维长度
    // 例: SeqArray<int>({3, 4, 5}) 构造 3x4x5 的三维数组
    explicit SeqArray(std::initializer_list<uint64_t> bounds);
    explicit SeqArray(const std::vector<uint64_t>& bounds);

    // DestroyArray - unique_ptr 自动销毁
    ~SeqArray() = default;

    // Value - 根据下标取值
    // 例: arr.get({1, 2, 3}) 取第[1][2][3]个元素
    const T& get(std::initializer_list<uint64_t> indices) const;

    // Assign - 根据下标赋值
    // 例: arr.set({1, 2, 3}, value)
    void set(std::initializer_list<uint64_t> indices, const T& value);

    // 维数
    [[nodiscard]] uint64_t dimensions() const;

    // 第i维的长度

    [[nodiscard]] uint64_t bound(uint64_t dim) const;

    //最多能够存储的元素个数
    [[nodiscard]] uint64_t total_size() const;

private:
    std::unique_ptr<T[]> _data;         // 扁平一维数组
    std::vector<uint64_t> _bounds;      // 各维长度 {b1, b2, ..., bn}    std::vector<uint64_t> _constants;   // 映射常量 ci = b(i+1)*b(i+2)*...*bn, cn=1    uint64_t _total;                    // 总元素数 = b1*b2*...*bn
    // 根据多维下标计算一维偏移量
    // offset = j1*c1 + j2*c2 + ... + jn*cn
    uint64_t _offset(const std::vector<uint64_t>& indices) const;

    // 初始化 _constants 和 _total    void _init_constants();
};

// ==================== 实现 ====================
template<typename T>
SeqArray<T>::SeqArray(std::initializer_list<uint64_t> bounds) :SeqArray(std::vector<uint64_t>(bounds)) {}

template<typename T>
SeqArray<T>::SeqArray(const std::vector<uint64_t>& bounds) : _bounds(bounds) {
    _init_constants();
    _data = std::make_unique<T[]>(_total);
}

template<typename T>
const T& SeqArray<T>::get(std::initializer_list<uint64_t> indices) const {
    return _data[_offset(indices)];
}

template<typename T>
void SeqArray<T>::set(std::initializer_list<uint64_t> indices, const T& value) {
    _data[_offset(indices)] = value;
}

template<typename T>
uint64_t SeqArray<T>::dimensions() const {
    return _bounds.size();
}

template<typename T>
uint64_t SeqArray<T>::bound(uint64_t dim) const {
    return _bounds[dim];
}

template<typename T>
uint64_t SeqArray<T>::total_size() const {
    return _total;
}

template<typename T>
uint64_t SeqArray<T>::_offset(const std::vector<uint64_t>& indices) const {
    if (indices.size() != _bounds.size())
        throw std::invalid_argument("维度不匹配");
    uint64_t offset = 0;
    for (uint64_t i = 0; i < indices.size(); ++i) {
        if (indices[i] >= _bounds[i])
            throw std::out_of_range("索引越界");
        offset += indices[i] * _constants[i];
    }
    return offset;
}

template<typename T>
void SeqArray<T>::_init_constants() {
    // _constants[i] = b(i+1) * b(i+2) * ... * bn
    // _constants[n-1] = 1    uint64_t n = _bounds.size();

    _constants.resize(n);
    _constants[n - 1] = 1;
    //从n-1维到0维
    for (uint64_t i = n - 1; i > 0; --i) {
        _constants[i - 1] = _bounds[i] * _constants[i];
    }

    _total = _bounds[0] * _constants[0];



}

#endif //DATA_STRUCTURE_LEARN_SEQ_ARRAY_HPP
```
