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


# 参考资料

[GLFW 官网](https://www.glfw.org/)
