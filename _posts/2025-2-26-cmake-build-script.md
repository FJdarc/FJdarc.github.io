# 一个用python写的CMake构建脚本

---

[github](https://github.com/FJdarc/CMakeBuildScript)

需要安装gcc和g++，cmake和python。

进入CMakeLists.txt文件夹

1.Windows

打开Windows CMD，执行下面的命令

`curl -O https://raw.githubusercontent.com/FJdarc/CMakeBuildScript/master/Cbs.py && .\Cbs.py -h`

2.Linux

`curl -O https://raw.githubusercontent.com/FJdarc/CMakeBuildScript/master/Cbs.py && chmod +x ./Cbs.py && .\Cbs.py -h`

---

即可得到Cbs脚本的帮助信息如下

```bash
usage: Cbs.py [-h] [{x64,x86}] [{d,r}] [{st,sh}] [program_name]

CMake项目构建工具

positional arguments:
  {x64,x86}     目标架构:
                x64 - 64位架构 (默认)
                x86 - 32位架构
  {d,r}         构建类型:
                d - 调试版本 (默认)
                r - 发布版本
  {st,sh}       库类型:
                st - 静态库 (默认)
                sh - 动态库/DLL
  program_name  指定输出程序名称 (默认使用当前目录名)

options:
  -h, --help    显示帮助信息并退出
```

