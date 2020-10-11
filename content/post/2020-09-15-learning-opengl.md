---
title: OpenGL 学习总结 (The Cherno OpenGL 系列教程)
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


# 使用 Vertex Buffer

Vertex Buffer 用来定义顶点数据，如绘制一个平面的三角形，我们需要定义三个顶点的位置，(x1, y1)、 (x2, y2)、 (x3, y3)。然后调用 `glDrawArrays` 进行绘制即可。

使用 Vertex Buffer 的时候，我们需要把它们绑定到 `GL_ARRAY_BUFFER` 上。

```cpp
float positions[] = {-0.5f, -0.5f, 0.0f, 0.5f, 0.5f, -0.5f};
unsigned int buffer;
// 创建 buffer
glGenBuffers(1, &buffer);
// 绑定当前 buffer
glBindBuffer(GL_ARRAY_BUFFER, buffer);
// 为当前 buffer 填充数据
glBufferData(GL_ARRAY_BUFFER, sizeof(positions) / sizeof(*positions) * sizeof(float), positions, GL_STATIC_DRAW);

// 启用顶点数据
glEnableVertexAttribArray(0);
// 指定顶点的属性
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);

// 根据当前状态啊进行绘制
glDrawArrays(GL_TRIANGLES, 0, 3);

// 删除 buffer
glDeleteBuffers(1, &buffer);
```

注意，MacOS 默认会用 `Legacy Profile`，对应的 OpenGL 版本是 2.1，GLSL 则是 1.20。

我们可以通过如下的方式去测试自己当前使用的 OpenGL 版本：

```c++
std::cout << glGetString(GL_VERSION) << std::endl;
// MacOS 下会输出
// 2.1 ATI-3.10.15
```

如果想要用更高版本的 OpenGL，那么我们需要改用 `Core Profile`：

```cpp
if (!glfwInit())
    return -1;

// 在创建 window 之前添加如下代码
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);

// 这时候调用 glGetString(GL_VERSION) 会输出
// 4.1 ATI-3.10.15
```

另外，在 MacOS 中我们还需要启用 Vertex Array Object 才能使用自定义的 shader：

```cpp
unsigned int vao;
glGenVertexArrays(1, &vao);
glBindVertexArray(vao);

glEnableVertexAttribArray(0);
// ...
```



# 使用 Index Buffer

由于 OpenGL 最基础的图元是三角形，因此如果我们想要绘制复杂图形的话，我们就需要通过绘制多个三角面来实现。

假设我们想要绘制一个矩形，那么我们就需要定义两个三角面，即 6 个顶点坐标，这种情况下就会出现顶点数据重复，数据冗余的情况。其实我们可以定义 4 个顶点坐标，然后告诉 OpenGL 如何使用这些顶点去绘制三角形即可。

和 Vertex Buffer 类似，我们需要把 Index Buffer 是绑定到 `GL_ELEMENT_ARRAY_BUFFER` 上，然后用 `glDrawElements` 来代替 `glDrawArrays`。


```cpp
// ... bind vertex buffers

float positions[] = {-0.5f, -0.5f, 0.5f, -0.5f, 0.5f,  0.5f, -0.5f,  0.5f};
unsigned int indices[] = {
    0, 1, 2, // 第一个三角形使用的顶点
    0, 2, 3  // 第二个三角形使用的顶点
};
unsigned int ibo;
glGenBuffers(1, &ibo);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ibo);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices) / sizeof(*indices) * sizeof(unsigned int), indices, GL_STATIC_DRAW);

glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr);
```



# 使用 Shader

```cpp
int compileShader(unsigned int type, const std::string& source) {
    unsigned int id = glCreateShader(type);
    const char* src = source.c_str();
    glShaderSource(id, 1, &src, nullptr);
    glCompileShader(id);
    
    return id;
}

int createShader(const std::string& vertexShader, const std::string& fragmentShader) {
    unsigned int program = glCreateProgram();
    unsigned int vs = compileShader(GL_VERTEX_SHADER, vertexShader);
    unsigned int fs = compileShader(GL_FRAGMENT_SHADER, fragmentShader);
    glAttachShader(program, vs);
    glAttachShader(program, fs);
    glLinkProgram(program);
    glValidateProgram(program);
    
    glDeleteShader(vs);
    glDeleteShader(fs);
    
    return id;
}

std::string vertexShader = "...";
std::string fragmentShader = "...";
unsigned int shader = createShader(vertexShader, fragmentShader);
glUseProgram(shader);
```


## 从文件中获取 Shader 内容

我们可以把 Shader 的内容写到单独的文件中，然后通过读取文件的方式去获得 Shader 的内容。

这里面有两个派系，一种是把 Vertex Shader 和 Fragment Shader 保存到两个文件中，另一种则是把它们都保存到同一个文件中，然后通过某些标记去区分。

Cherno 的教程中是把它们都保存到一个文件中，用 `#shader vertex` 和 `#shader fragment` 去区分 Vertex Shader 和 Fragment Shader 的代码。

```glsl
// in res/shaders/basic.shader

#shader vertex
#version 330 core

void main() {
    // ...
}

#shader fragment
#version 330 core

void main() {
    // ...
}
```


```cpp
// in src/main.cpp
#include <fstream>
#include <string>
#include <sstream>

struct ShaderProgramSource {
    std::string vertexSource;
    std::string fragmentSource;
}

ShaderProgramSource parseShader(const std::string& filepath) {
    enum class ShaderType {
        NONE = -1,
        VERTEX = 0,
        FRAGMENT = 1
    };
    std::ifstream stream(filepath);
    std::string line;
    std::stringstream ss[2];
    ShaderType type = ShaderType::NONE;
    
    while (getline(stream, line)) {
        if (line.find("#shader") != std::string::npos)  {
            if (line.find("vertex") != std::string::npos)  {
                type = ShaderType::VERTEX; 
            } else if (line.find("fragment") != std::string::npos) {
                type = ShaderType::FRAGMENT;
            }
        } else {
            ss[(int)type] << line << '\n';
        }
    }

    return { ss[0].str(), ss[1].str() };
}

ShaderProgramSource source = parseShader("../res/shaders/basic.shader");
unsigned int shader = createShader(source.vertexSource, source.fragmentSource);
glUseProgram(shader);
```


# 补充知识

## OpenGL (Open Graphics Library)

OpenGL 是一套跨语言、跨平台的图形编程接口，它由许许多多的函数组成，这些函数是由显卡驱动来实现的，因此我们只需要下载并安装显卡驱动，就能使用 OpenGL 了。



## GLEW (OpenGL Extension Wrangler Library)

当我们在编写 OpenGL 程序的时候，编译器需要知道 OpenGL 各种函数的地址，才能正确编译出程序。但由于这些函数是在显卡驱动中实现的，而不同的平台有不同的显卡驱动，这就会导致一个问题：我们的代码不能跨平台运行。

GLEW 就是一个为 OpenGL 抹平平台间差异的库。它做的事情很简单：在运行时根据当前平台去找 OpenGL 的函数地址，实现方式如下：

```cpp
#define glGenBuffers GLEW_GET_FUN(__glewGenBuffers)
```



## GLFW (Graphics Library Framework)

由于 OpenGL 只关注渲染，因此它的规范中并没有包含如何获得、管理 OpenGL 上下文，也没有提供管理窗口、用户输入等的API，因此这些功能需要由其他库来提供。

GLFW 就是一个可以提供这些功能的 API 的库，它允许用户创建 OpenGL 上下文，定义窗口参数以及处理用户输入。



# 参考资料

[The Cherno OpenGL Series](https://www.youtube.com/playlist?list=PLlrATfBNZ98foTJPJ_Ev03o2oq3-GGOS2)

[OpenGL](https://zh.wikipedia.org/wiki/OpenGL)

[GLEW 官网](http://glew.sourceforge.net/)

[GLFW 官网](https://www.glfw.org/)

[docs.GL](http://docs.gl/)

