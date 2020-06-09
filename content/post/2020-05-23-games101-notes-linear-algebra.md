---
title: Games101 笔记 —— 线性代数
date: 2020-05-23T16:28:10+08:00
---

# 简介
本文《GAMES101-现代计算机图形学入门》系列教程的课程笔记，仅用于个人学习使用。


# 向量

向量是用来表示方向的量，它由两部分组成：大小、方向。从定义中可以看出，向量并不关心起始的绝对位置，它只关心朝向哪个方向，以及往该方向的长度。

## 方向

在数学上一般以 $\vec{AB}$ 表示由 A 指向 B 的向量，通常也会简化成 $\vec a$。

$$
\vec a = \vec {AB} = B - A
$$

## 长度

向量 $\vec a$ 的长度用 $||\vec{a}||$ 来表示。

## 单位向量

单位向量是指长度为 1 的向量，以 $\hat a$ （读作 a hat）来表示。

计算公式：

$$
\hat a = \frac{\vec a} {||\vec{a}||}
$$

## 向量加法

### 几何加法

在几何上，向量加法可以用三角形法则或平行四边形法则来表示。

### 算术加法

在算术上，直接把两个向量的各个坐标加起来即可。

$$ \vec a = (x_a, y_a) $$

$$ \vec b = (x_b, y_b) $$

$$ \vec a + \vec b = (x_a + x_b, y_a + y_b) $$


## 向量点乘 (Dot Product)

向量点乘的结果是一个数，这个结果在几何上和代数上有不同的意义。

### 几何意义

向量点乘在几何上的定义如下：

$$ \vec a \cdot \vec b = ||\vec a|| \cdot ||\vec b|| \cdot cos \theta $$

通常会有下面个变种形式：

$$ cos \theta = \frac{\vec a \cdot \vec b} {||\vec a|| \cdot ||\vec b ||} $$

当 $\vec a$ 和 $\vec b$ 为单位向量时：

$$ cos \theta = \vec a \cdot \vec b $$

点乘在几何上通常有以下这几种用途：

- 求两个向量的夹角（计算 $cos \theta$ 的反三角函数）。
- 求两个向量是同向还是背向（通过判断 $cos \theta$ 的正负）。
- 求一个向量在另一个向量上的投影。 证明：

$$ \vec {b_\bot} = k \cdot \hat a $$

$$ k = ||\vec {b_\bot}|| = \vec b \cdot cos \theta $$

- 分解向量（分解成一对相互垂直的向量的和）。



## 向量叉乘 （Cross Product）

向量叉乘通常写成如下的形式：

$$
\vec c = \vec a \times \vec b
$$

所得结果 $\vec c$ 是一个同时垂直于 $\vec a$ 和 $\vec b$ 的向量。

需要注意的是，向量叉乘并不满足交换律，因此有：
$$
\vec a \times \vec b = - \vec b \times \vec a
$$

叉乘通常有以下几种用途：

- 计算垂直向量。
- 计算 $\vec b$ 在 $\vec a$ 的左边还是右边（在 XY 平面上，根据 `z` 的正负判断）。
- 计算平面上的一点 `P` 是否位于三角面内部（$\vec{AB} \times \vec{AP}$、$\vec{BC} \times \vec{BP}$、$\vec{CA} \times \vec{CP}$ 的结果都在左边或都在右边）。


# 矩阵

矩阵由 m 行 n 列的数字组成，通常表示成 $m \times n$。

## 矩阵乘法

矩阵相乘通常表示成 $A \times B$，但两个矩阵必须满足一个条件才能相乘： A 的列数等于 B 的行数。

$$
A = (m \times n)
$$

$$
B = (n \times p)
$$

$$
A \times B = (m \times n)(n \times p) = (m \times p)
$$


相乘后得到一个新的矩阵，由 m 行 p 列组成。

其中第 i 行 j 列的元素等于 A 的第 i 行与 B 的第 j 列的点积。
 
需要注意的是，矩阵乘法**并不**满足交换律，即：

$$
A \times B \ne B \times A
$$

但矩阵乘法满足分配律和结合率，即：

$$
(A + B)C = AC + BC
$$

$$
(A \times B) \times C = A \times (B \times C)
$$

## 转置矩阵

转置矩阵是指行列互换的矩阵。

转置矩阵有一个特殊的性质：

$$
(AB)^T = B^TA^T
$$

## 矩阵与向量

矩阵可以与向量相乘，相乘时可以把向量当成是 $m \times 1$ 的矩阵。

点乘：

$$
\vec a \cdot \vec b = {\vec a}^T \cdot \vec b
$$

$$
\vec a \cdot \vec b = (x_a y_a z_a) \begin{pmatrix} x_b \\\ y_b \\\ z_b\end{pmatrix}
$$

$$
\vec a \cdot \vec b = (x_ax_b + y_ay_b + z_az_b)
$$


叉乘（A^* 为一个特殊的矩阵，称为 dual matrix）：

$$
\vec a \times \vec b = A^*\vec b
$$


# 参考资料
[Lecture 02 Review of Linear Algebra](https://www.bilibili.com/video/BV1X7411F744?p=2)
