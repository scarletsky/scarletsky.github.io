---
title: 深度变换总结
date: 2021-03-06T00:22:36+08:00
tags: [graphics, opengl, webgl]
---

# 渲染管线变换


# 投影变换与逆变换

投影矩阵分为透视矩阵和正交矩阵。

- 透视矩阵

<div>
$$
\begin{bmatrix}
{\frac {2n} {r-l}} & 0 & {\frac {r+l} {r-l}} & 0 \\
0 & {\frac {2n} {t-b}} & {\frac {t+b} {t-b}} & 0 \\
0 & 0 & {\frac {-(f + n)} {f - n}} & {\frac {-2fn} {f - n}} \\
0 & 0 & -1 & 0
\end{bmatrix}
$$
</div>

- 正交矩阵

<div>
$$
\begin{bmatrix}
{\frac {2} {r-l}} & 0 & 0 & -{\frac {r+l} {r-l}} \\
0 & {\frac {2} {t-b}} & 0 & -{\frac {t+b} {t-b}} \\
0 & 0 & {\frac {-2} {f-n}} & -{\frac {f+n} {f-n}} \\
0 & 0 & 0 & 1
\end{bmatrix}
$$
</div>

<div>

假设 view space 下的某一点的齐次坐标为 $ (X_{view}, Y_{view}, Z_{view}, 1.0) $ ，第三行第三列为 A，第三行第四列为 B，那么 clip space 下的 Z 可以表示成：

$$
Z_{clip} = A Z_{view} + B
$$

从 clip space 变换成 ndc space 时，需要执行透视除法，那么就有：
</div>

## 透视投影变换

<div>
$$
W_{clip} = -Z_{view}
$$

$$
Z_{ndc} = \frac {Z_{clip}}  {W_{clip}} = \frac {A Z_{view} + B} {-Z_{view}} = -A - \frac B {Z_{view}}
$$
</div>

## 正交投影变换

<div>
$$
W_{clip} = W_{view} = 1.0
$$

$$
Z_{ndc} = \frac {Z_{clip}} {W_{clip}} = A {Z_{view}} + B
$$
</div>


<div>
在 ndc space 中，$ Z_{ndc} $ 的取值范围为 [-1, 1]。当我们需要把它保存到 Z buffer 中时，需要把取值范围变换成 [0, 1]：

$$
Z_{buffer} = Z_{ndc} * 0.5 + 0.5
$$

</div>

这就是我们经常看的**深度图（Depth Map）** 了。


反过来说，我们也可以从深度图里面提取出 $ Z_{buffer} $，然后计算出其他坐标。

<div>

先把深度图中的值变换到 ndc space 中：

$$
Z_{ndc} = Z_{buffer} * 2.0 - 1.0
$$

再把 ndc space 变换到 clip space：

$$
Z_{clip} = Z_{ndc} * W_{clip}
$$

但这个过程要用到 $ Z_{view} $ 和 $ W_{clip} $，不同的投影矩阵会有不同的计算过程。

</div>

## 透视投影逆变换

<div>

在透视投影变换时，$ W_{clip} $ 和 $ Z_{view} $ 有如下的关系：

$$
W_{clip} = -Z_{view}
$$

从前面的式子可以知道 Z 在 ndc space 和 view space 下有如下的关系：

$$
Z_{ndc} = -A - \frac B {Z_{view}}
$$

那么我们可以把这个式子化简：

$$
Z_{view} = \frac {-B} {A+Z_{ndc}}
$$

把 A 和 B 都展开，就可以得到：

$$
Z_{view} = \frac {2fn} {Z_{ndc}(f-n) - (f+n)}
$$

我们还可以用 $ Z_{buffer} $ 代替 $ Z_{ndc} $，这样就可以得到：


$$
Z_{view} = \frac {2fn} {(2Z_{buffer} - 1)(f-n) - (f+n)} = \frac {fn} {Z_{buffer}(f-n) - f}
$$

</div>

## 正交投影逆变换

<div>
在正交投影中，$ W_{clip} = W_{view} = 1.0 $，因此透视除法并没有影响原来的 Z。


$$
Z_{ndc} = A Z_{view} + B
$$

化简得：

$$
Z_{view} = \frac {Z_{ndc} - B} {A}
$$

把 A 和 B 都展开：

$$
Z_{view} = \frac {Z_{ndc}(f-n)+(f+n)} {-2}
$$

再用 $ Z_{buffer} $ 代替 $ Z_{ndc} $：

$$
Z_{view} = \frac {(2Z_{buffer} - 1)(f-n)+(f+n)} {-2} = Z_{buffer}(n-f) - n
$$

</div>


<div>

这样，我们就可以从 $ Z_{ndc} $ 或者 $ Z_{buffer} $ 中计算出 $ Z_{view} $，然后就可以计算出 $ W_{clip} $ 了。

假设 $ P_{43} $ 表示投影矩阵的第四行第三列，$ P_{44} $ 表示第四行第四列，根据矩阵乘法的计算我们可以得到：

$$
W_{clip} = P_{43} * Z_{view} + P_{44}
$$

当矩阵为透视矩阵的时候有 $ W_{clip} = -Z_{view} $；矩阵为正交矩阵的时候有 $ W_{clip} = 1.0 $。

有了 $ W_{clip} $ 后，我们不仅能计算出 $ Z_{clip} $，我们也可以计算出当前 fragment 在 clip space 和 view space 下的坐标。

</div>


# 深度与线性深度

这几个是经常出现并且很容易混淆的概念：

- **深度**指的是最终在 Z buffer 中的值，取值范围是 $ [0, 1] $。 当使用正交投影时，深度是线性变化的；当使用透视投影时，深度是非线性变化的。

- **线性深度**指的是还原成线性变化的深度。


# 应用

深度非常常用，除了做深度测试之外，它还可以用来做各种各样的事情。我们来看看下面这些例子：

## 提取 view space 下的坐标

<div>

假设 $ X_{screen} $、$ Y_{screen} $、$ Z_{screen}$、分别表示 screen space 下的坐标， $ P_{ndc} $ 、$ P_{clip} $ 、$ P_{view} $ 分别表示 P 点在 ndc space、clip space、view space 下的位置，$ W_{clip} $ 表示 P 点在齐次坐标下的 W 分量在 clip space 下的值， $ M_{projection} $、 $ M_{projection}^{-1}$ 、$ M_{view} $、$ M_{view}^{-1} $ 分别表示投影矩阵、投影矩阵的逆矩阵、视图矩阵、视图矩阵的逆矩阵。

</div>

<div>
因为：

$$
P_{clip} = M_{projection} * P_{view}
$$

$$
P_{ndc} = P_{clip} / W_{clip}
$$

所以：

$$
P_{ndc} = ((X_{screen}, Y_{screen}, Z_{screen}) * 2.0 - 1.0, 1.0)
$$

$$
P_{clip} = P_{ndc} * W_{clip}
$$

$$
P_{view} = M_{projection}^{-1} * P_{clip}
$$
</div>

## 提取 view space 下的法线

```glsl
vec3 getViewNormal(const in vec3 viewPosition) {
    return normalize(cross(dFdx(viewPosition), dFdx(viewPosition)));
}
```

# 总结



# 备注


# 参考资料

[OpenGL Projection Matrix](http://www.songho.ca/opengl/gl_projectionmatrix.html)

[Real depth in OpenGL / GLSL](http://web.archive.org/web/20130416194336/http://olivers.posterous.com/linear-depth-in-glsl-for-real)

[Reconstructing Position From Depth](https://therealmjp.github.io/posts/reconstructing-position-from-depth/)
