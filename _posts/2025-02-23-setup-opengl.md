---
layout: mypost
title: 设置OpenGL环境并用C++绘制一个窗口
categories: [OpenGL, 学习]
---
https://github.com/glfw/glfw/releases/download/3.4/glfw-3.4.bin.WIN64.zip

https://github.com/glfw/glfw/releases/download/3.4/glfw-3.4.bin.WIN32.zip

Visual Studio创建CMake项目

在项目根目录创建文件夹Dependencies



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



```c++
// OpenGL.h: 标准系统包含文件的包含文件
// 或项目特定的包含文件。

#pragma once

#include <GLFW/glfw3.h>

// TODO: 在此处引用程序需要的其他标头。
```

