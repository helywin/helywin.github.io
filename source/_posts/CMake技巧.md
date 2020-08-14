---
title: CMake技巧
date: 2020-7-31 09:41:20
tags:
  - CMake
---

## 1 判断CMake编译环境

- 编译类型`CMAKE_BUILD_TYPE`

可取Debug, Release, RelWithDebInfo, MinSizeRel等等预设值

```cmake
if (CMAKE_BUILD_TYPE MATCHES Debug)
	#do some thing
endif()
```

- 系统环境`CMAKE_SYSTEM_NAME`

代表当前系统的类型, 值有ANDROID, APPLE, IOS, UNIX, WIN32, WINCE, WINDOWS_PHONE等

可以直接对这些值进行条件判断来确定

```cmake
if (UNIX)
	#cond1
elseif(WIN32)
	#cond2
endif()
```

- 编译工具环境

编译环境包括MSVC, MINGW, BORLAND, 等

可直接用if判断

msvc版本可通过MSVC10, MSVC11, MSVC12, MSVC14, MSVC60, MSVC70, MSVC71, MSVC80, MSVC90, MSVC_TOOLSET_VERSION, MSVC_VERSION等判断

<!-- more -->

## 2 编译器设置

- C/C++标准

```cmake
# 全局
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_C_STANDARD 11)
```

- 编译器flag

主要靠修改`CMAKE_CXX_FLAG_<BUILD_TYPE>`来进行修改, 也可以直接修改所有类型的, 或者通过判断编译类型来配置

```cmake
set(DEFINES "-Wunused-parameter")
set(CMAKE_CXX_FLAGS_DEBUG "-pipe -O2 -std=gnu++11 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_C_FLAGS_DEBUG "-pipe -O2 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_EXE_LINKER_FLAGS_DEBUG "-fno-pie")

set(CMAKE_CXX_FLAGS_RELEASE "-pipe -O2 -std=gnu++11 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_C_FLAGS_RELEASE "-pipe -O2 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "-fno-pie")
```

- 特殊的编译器设置

  - sanitize (msvc目前只支持32位)

    检查内存泄露和内存其他问题

    ```cmake
    list(APPEND CMAKE_CXX_FLAGS_DEBUG -fsanitize=address -g)
    ```

    配置后运行程序时发生内存问题程序会立马中断退出, 并且打印内存问题

  - /utf-8 (msvc)

    强制源码使用utf-8, 避免代码到了其他平台乱码

    ```cmake
    list(APPEND CMAKE_CXX_FLAGS_DEBUG /utf-8)
    ```

  - -DUNICODE (msvc)

    决定windows下面使用标准字符api还是宽字符api

    ```c++
    #ifdef UNICODE
    #define ShellExecute  ShellExecuteW
    #else
    #define ShellExecute  ShellExecuteA
    #endif // !UNICODE
    ```

  - /SUBSYSTEM:WINDOWS
  
    更改软件启动入口, 默认是/SUBSYSTEM:CONSOLE, 会先启动命令行再启动其他ui, 而更改后命令行将不再启动, 所有打印信息也将会看不到, 只有通过x64dbg这种调试软件才能看到
  
  - /OPT:REF /OPT:ICF
  
    添加这两个参数release模式下编译也会生成pdb文件

