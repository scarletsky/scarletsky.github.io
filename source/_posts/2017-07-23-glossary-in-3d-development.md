---
title: 3D 开发中常见的术语
date: 2017-07-23 16:06:39
category: technology
tags: [3d]
---

## 简介
在 3D 开发领域，我们会经常看到一些术语，有些是和数学相关的，有些是和模型相关的，也有一些是和程序相关的。本文是我对 3D 开发中常见术语的总结。

## 数学
数学是 3D 开发的基石，无论你想要操作物体移动，还是在屏幕中显示一个 3D 物体，都涉及到数学计算。在 3D 开发中，最常见的数学工具是向量、矩阵、四元数。

### 向量(Vector)
向量是既有大小又有方向的量。下面是常见的类型：

- 二维向量(Vec2)
  主要用来记录平面相关的值，如鼠标事件/触摸事件的发生的坐标，二维物体坐标等。
  它有两个分量，分别是 `x` 和 `y`。

- 三维向量(Vec3)
  主要用来记录空间坐标，如三维物体的位置。也可以用来表示方向，如一个平面的法线方向。
  它有三个分量，分别是 `x` 和 `y` 和 `z`。

- 四维向量(Vec4)
  这种向量比较特殊，因为你可以把它当成齐次坐标来使用。
  这时候它的四个分量分别是：`x`、`y`、`z`、`w`。
  当 `w` 等于 1 时，它表示空间中的一点。
  当 `w` 不等于 1 时，它表示的是方向。

  你也可以用它来记录颜色的值，这时候它的四个分量就是 `r`、`g`、`b`、`a`。


计算方式：

- 加法
- 减法
- 点乘
- 叉乘


### 矩阵(Matrix)

### Quaternion

### EulerAngles



## 3D模型

### Mesh

### Material

### Texture

### Axis Align Bounding Box

### Orient Bounding Box

### Ray


## 程序

### Vertex

### Shading

### Shader

### Draw Call

## 参考资料
[3D游戏开发术语](https://jmonkeyengine.github.io/wiki/jme3/terminology_zh.html)
[OpenGL 教程第三课：矩阵](http://www.opengl-tutorial.org/cn/beginners-tutorials/tutorial-3-matrices/)
