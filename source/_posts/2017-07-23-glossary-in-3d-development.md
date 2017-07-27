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

向量常见的操作有以下几种：

- 加法
- 减法

- 规范化(normalize)
  即把该向量变成单位向量(长度为1)。

- 点乘
  计算公式为：
```
a.dot(b) = a.length() * b.length() * Math.cos(theta)
```
  点乘的结果是一个标量，通常用来求向量的在另一个向量上的投影长度，也可以用来求两向量之间的夹角。
  另外，当两向量都是单位向量时，我们可以简化上面的公式：
```
a.dot(b) = Math.cos(theta)
theta = Math.acos(a.dot(b));
```

- 叉乘
  叉乘的结果是一个向量，该向量会垂直于另外两个变量。它满足下面公式：
```
a.cross(b) = c
c.length() = a.length() * b.length() * Math.sin(theta)
```
  我们知道，三个不在同一直线上的点可以构成一个平面，我们可以通过用这三个点计算两个向量，然后再计算这两个向量的叉乘，结果就是这个平面的法线了。


### 矩阵(Matrix)


### 四元数(Quaternion)


### 欧拉角(EulerAngles)



## 3D 模型
任何一个 3D 模型都是由多个简单的几何形状组成的，而这些几何形状又是通过三角形来组成的。


### 顶点(Vertex)
在三维空间中，一个孤立的点是普通的点。如果一个点是用来构成多边形的，那么该点就被称为顶点。 

### 网格(Mesh)
网格是一种复杂的形状，它由多个顶点按照某个规律连接而成，换句话说，一个网格包含了多个顶点。
一个模型可以包含多个网格。

### 材质(Material)
材质管理着颜色、明暗、光泽等信息。当一个网格加上一个材质之后，它就可以在屏幕上显示特定的「色彩」了。 一个网格只有一种材质，而一种材质可以应用在多个网格上。

### 贴图(Texture)
贴图是指把一张普通的图片贴到材质的表面。每种材质都可以有多种类型的贴图，如漫反射贴图、法线贴图、高光贴图、光照贴图 等。材质除了这些贴图之外，还有其他属性，可以用来调节这些贴图的属性。
换句话说，每种材质都可以包含多种贴图。

### UV 贴图

### 轴向包围盒(Axis Align Bounding Box)
轴向包围盒简称 AABB，是指包含一个模型，且边平行于坐标轴的最小六面体。
请看看下面两张图：
![](http://7tebgv.com1.z0.glb.clouddn.com/images/aabb.png)
![](http://7tebgv.com1.z0.glb.clouddn.com/images/aabb2.png)
AABB 就是在模型最外面的由细线构成包围盒，它会随着模型旋转而变化。
实际上，AABB 并不可见，上图中的只是为了让我们更好理解才把包围盒画上去的。
另外，第一张图中为了让我们更好看见 AABB，特意把 AABB 调大了一点点，实际上该模型的 AABB 和所有边都是重合的。

### 有向包围盒(Orient Bounding Box)
有向包围盒简称 OBB，它是包含模型且相对于坐标轴方向任意的最小的长方体。
看看下面两张图：
![](http://7tebgv.com1.z0.glb.clouddn.com/images/obb.png)
![](http://7tebgv.com1.z0.glb.clouddn.com/images/obb2.png)
白色线框表示的就是 OBB，而蓝色线框表示的则是 AABB。
如果想看更直观的对比，可以看看这个视频：[AABB vs OBB](https://www.youtube.com/watch?v=HYO5Pthe3TE)。


## 程序

### Vertex Buffer

### Shading

### Shader

### Draw Call

## 参考资料
[3D游戏开发术语](https://jmonkeyengine.github.io/wiki/jme3/terminology_zh.html)
[OpenGL 教程第三课：矩阵](http://www.opengl-tutorial.org/cn/beginners-tutorials/tutorial-3-matrices/)
