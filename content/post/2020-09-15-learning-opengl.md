---
title: Learning OpenGL
date: 2020-09-15T09:10:50+08:00
draft: true
---


# 安装

```
brew install glfw glew
```


# 配置

在 `CMakeLists.txt` 中添加相关配置：

```cmake
set(TARGET_NAME HelloOpenGL)

find_package(OpenGL REQUIRED)
find_package(glfw3 3.3 REQUIRED)
find_package(GLEW REQUIRED)

add_executable(${TARGET_NAME} src/main.cpp)
target_link_libraries(${TARGET_NAME} OpenGL::GL GLEW::GLEW glfw)
```

然后在代码中引用 `glew` 和 `glfw`：

```c++
#include <GL/glew.h>
#include <GLFW/glfw3.h>
```

需要注意的是，我们需要确保 `glew` 比 `glfw` 更早引用，否则会出现如下报错：

```
error: gl.h included before glew.h
```


# 状态机

OpenGL 是一个状态机，我们通过调用它的 API 来更新当前的状态，然后根据当前状态进行绘制。


## 使用 Buffer

```c++
float positions[6] = {-0.5f, -0.5f, 0.0f, 0.5f, 0.5f, -0.5f};
unsigned int buffer;
// 创建 buffer
glGenBuffers(1, &buffer);
// 绑定当前 buffer
glBindBuffer(GL_ARRAY_BUFFER, buffer);
// 为当前 buffer 填充数据
glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW);

// 启用顶点数据
glEnableVertexAttribArray(0);
// 指定顶点的属性
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);

// 根据当前状态啊进行绘制
glDrawArrays(GL_TRIANGLES, 0, 3);

// 删除 buffer
glDeleteBuffers(1, &buffer);
```


# 补充知识

## OpenGL (Open Graphics Library)

OpenGL 是一套跨语言、跨平台的图形编程接口，它由许许多多的函数组成，这些函数是由显卡驱动来实现的，因此我们只需要下载并安装显卡驱动，就能使用 OpenGL 了。


## GLEW (OpenGL Extension Wrangler Library)

当我们在编写 OpenGL 程序的时候，编译器需要知道 OpenGL 各种函数的地址，才能正确编译出程序。但由于这些函数是在显卡驱动中实现的，而不同的平台有不同的显卡驱动，这就会导致一个问题：我们的代码不能跨平台运行。

GLEW 就是一个为 OpenGL 抹平平台间差异的库。它做的事情很简单：在运行时根据当前平台去找 OpenGL 的函数地址。


## GLFW (Graphics Library Framework)

由于 OpenGL 只关注渲染，因此它的规范中并没有包含如何获得、管理 OpenGL 上下文，也没有提供管理窗口、用户输入等的API，因此这些功能需要由其他库来提供。

GLFW 就是一个可以提供这些功能的 API 的库，它允许用户创建 OpenGL 上下文，定义窗口参数以及处理用户输入。



# 参考资料

[The Cherno OpenGL Series](https://www.youtube.com/playlist?list=PLlrATfBNZ98foTJPJ_Ev03o2oq3-GGOS2)

[OpenGL](https://zh.wikipedia.org/wiki/OpenGL)

[GLEW 官网](http://glew.sourceforge.net/)

[GLFW 官网](https://www.glfw.org/)

[docs.GL](http://docs.gl/)

