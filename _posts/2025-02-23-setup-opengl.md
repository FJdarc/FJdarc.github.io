---
layout: mypost
title: 设置OpenGL环境并用C++绘制一个窗口
categories: [OpenGL, 学习]
---
#### GLFW

GLFW 是一个开源的跨平台库，用于在桌面环境中开发 OpenGL、OpenGL ES 和 Vulkan 应用程序。它提供了一套简单的 API 用于创建窗口、上下文和绘图表面，并接收用户输入和事件处理。

进入[GLFW下载页面](https://www.glfw.org/download.html)，选择Windows pre-compiled binaries右侧的64-bit Windows binaries和32-bit Windows binaries下载。

![Windows pre-compiled binaries](Windows pre-compiled binaries.png)

#### CMake in Visual Studio

在VS中创建CMake新项目，配置如下

![New CMake Config](New CMake Config.png)

创建之后，在项目根目录新建文件夹如下

![Dependencies Tree 1](Dependencies Tree 1.png)

将下载的64-bit Windows binaries和32-bit Windows binaries压缩包解压，移动64或者32里面的include文件夹到Dependencies\GLFW\下，选择对应版本的lib-vc版本文件夹，例如我是VS2022，就移动32里面的lib-vc2022到Dependencies\GLFW\WIN32，移动64里面的lib-vc2022到Dependencies\GLFW\WIN64。

移动完毕后的Dependencies文件夹如下

![Dependencies Tree 2](Dependencies Tree 2.png)

#### 代码部分

OpenGL\OpenGL\CMakeList.txt

```cmake
# CMakeList.txt: OpenGL 的 CMake 项目，在此处包括源代码并定义
# 项目特定的逻辑。
#

# 将源代码添加到此项目的可执行文件。
add_executable (OpenGL "OpenGL.cpp" "OpenGL.h")

if (CMAKE_VERSION VERSION_GREATER 3.12)
  set_property(TARGET OpenGL PROPERTY CXX_STANDARD 20)
endif()

target_include_directories(OpenGL PRIVATE
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_SOURCE_DIR}/Dependencies/GLFW/include  # GLFW 的头文件目录
)

set(GLFW_LIB64_DIR ${CMAKE_SOURCE_DIR}/Dependencies/GLFW/WIN64/lib-vc2022)
set(GLFW_LIB32_DIR ${CMAKE_SOURCE_DIR}/Dependencies/GLFW/WIN64/lib-vc2022)
if (CMAKE_SIZEOF_VOID_P EQUAL 8)  # 64 位
    set(GLFW_LIBRARY ${GLFW_LIB64_DIR}/glfw3.lib)
else()  # 32 位
    set(GLFW_LIBRARY ${GLFW_LIB32_DIR}/glfw3.lib)
endif()

target_link_libraries(OpenGL PRIVATE ${GLFW_LIBRARY} opengl32)

# TODO: 如有需要，请添加测试并安装目标。
```

OpenGL\OpenGL\OpenGL.cpp

```c++
// OpenGL.cpp: 定义应用程序的入口点。
//

#include "OpenGL.h"

int main(void)
{
    GLFWwindow* window;

    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);

        glBegin(GL_TRIANGLES);
        glVertex2f(-0.5f, -0.5f);
        glVertex2f( 0.0f,  0.5f);
        glVertex2f( 0.5f, -0.5f);
        glEnd();

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

OpenGL\OpenGL\OpenGL.h

```c++
// OpenGL.h: 标准系统包含文件的包含文件
// 或项目特定的包含文件。

#pragma once

#include <GLFW/glfw3.h>

// TODO: 在此处引用程序需要的其他标头。
```



**运行之后的效果是一个窗口当中显示一个三角形**
