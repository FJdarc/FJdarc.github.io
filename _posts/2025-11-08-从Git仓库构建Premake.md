---
layout: mypost
title: 从Git仓库构建Premake
categories: [Build, Premake]
---

## 1. 获取源码
```bash
git clone https://github.com/premake/premake-core.git
````

## 2. 引导（Bootstrap）

打开工具“x64 Native Tools Command Prompt for VS”
```bash
cd premake-core
```

```bash
nmake -f Bootstrap.mak windows
```

执行完成后，会生成一个可用的可执行文件在：

```
premake-core/bin/release/premake5.exe
```

## 3. 生成正式工程文件

根据你使用的工具集选择命令：

```bash
bin/release/premake5 vs2022   # 生成 Visual Studio 2022 工程
# 或
bin/release/premake5 gmake2   # 生成 GNU Makefile
```
