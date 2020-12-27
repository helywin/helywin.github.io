---
title: Qt CMake模板
date: 2018-12-16 11:41:40
tags:
  - Qt
  - CMake
excerpt: CMake管理Qt项目的模板和工程项目结构
index_img: https://s3.ax1x.com/2020/12/27/r5NTKg.png
---

按照如下目录结构建立好

```
├─assets    #图片
├─build        #构建目录
├─debug        #debug构建后的二进制文件
├─doc        #文档
├─lib        #第三方库
├─release    #release构建后的二进制文件
├─res        #Qt的qrc文件和rc文件
└─src        #源码目录
```

工程目录下的`CMakeLists.txt`文件如下：

```cmake
cmake_minimum_required(VERSION 3.12)
project(xxxx)

add_subdirectory(src)
```

`src`目录下`CMakeLists.txt`文件如下：

<!-- more -->

```cmake
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

find_package(Qt5Core CONFIG REQUIRED)
find_package(Qt5Widgets CONFIG REQUIRED)
find_package(Qt5Gui CONFIG REQUIRED)
find_package(Qt5OpenGL CONFIG REQUIRED)
#find_package(Qt5PrintSupport CONFIG REQUIRED)
#find_package(Qt5TextToSpeech CONFIG REQUIRED)
set(CMAKE_CXX_COMPILER g++)
set(CMAKE_C_COMPILER gcc)
set(DEFINES "-DUNICODE -D_UNICODE -DWIN32 -DQT_DEPRECATED_WARNINGS")
set(CMAKE_C_FLAGS "-fno-keep-inline-dllexport -fopenmp -march=i686 -mtune=core2 -Wa,-mbig-obj -O2 -Wall -W -Wextra ${DEFINES}")
set(CMAKE_CXX_FLAGS "-fno-keep-inline-dllexport -fopenmp -O2 -g -Wall -W -Wextra -fexceptions -mthreads ${DEFINES}")

if (CMAKE_BUILD_TYPE MATCHES Release)
    message("release compile!!")
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE "-Wl,-s -Wl,-subsystem,windows -mthreads")
    add_definitions(-DQT_NO_DEBUG)
    set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/release)
endif (CMAKE_BUILD_TYPE MATCHES Release)

if (CMAKE_BUILD_TYPE MATCHES Debug)
    message("debug compile!!")
    set(CMAKE_EXE_LINKER_FLAGS_DEBUG "-mthreads")
    set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/debug)
endif (CMAKE_BUILD_TYPE MATCHES Debug)

aux_source_directory(${PROJECT_SOURCE_DIR}/src SRC)
include_directories(${PROJECT_SOURCE_DIR}/src)

#set(UI ${PROJECT_SOURCE_DIR}/res/ui.qrc)

add_executable(${PROJECT_NAME} ${SRC})

target_link_libraries(${PROJECT_NAME}
        Qt5Core
        Qt5Widgets
        Qt5Gui
        Qt5OpenGL)
```

具体作用不解释，可以参考CMake手册和Qt手册关于CMake的部分

2019/4/21 补充：

1. 可以通过CMAKE_OS_NAME来判断用来编译的是什么环境

2. 从Qt官网下载的安装包CMake会找不到库的路径，可以添加以下一句解决
   
   ```cmake
   set(CMAKE_PREFIX_PATH qt-path/gcc_64)
   ```
   
   其中qt-path为qt库的路径

3. 链接时把库名改为如下方式可以减少include目录的复杂程度
   
   ```cmake
   target_link_libraries(${PROJECT_NAME}
           Qt5::Core
           Qt5::Widgets
           Qt5::Gui
           Qt5::OpenGL)
   ```

### 2019/4/28 补充

1. CMake可以 执行shell命令并且获取输出和返回值

```cmake
find_program(GIT git)

if ("${GIT}" STREQUAL "GIT-NOTFOUND")
    message(WARNING "找不到git程序,无法更新git提交记录")
    set(VERSION_REVISION "找不到提交号")
    set(VERSION_BRANCH "找不到提交分支")
else ()
    unset(GIT_VERSION_NUM CACHE)
    unset(GIT_BRANCH CACHE)
    message("找到git,读取分支信息")
    execute_process(
            COMMAND git log -1 --format=%H
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
            OUTPUT_VARIABLE GIT_COMMIT
            OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE GIT_COMMIT_RESULT
    )
    if (NOT GIT_COMMIT_RESULT EQUAL 0)
        message(FATAL_ERROR "找不到git提交的id号")
    else ()
        message("提交号 ${GIT_COMMIT}")
    endif ()

    execute_process(
            COMMAND git symbolic-ref --short HEAD
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
            OUTPUT_VARIABLE GIT_BRANCH
            OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE GIT_BRANCH_RESULT
    )
    if (NOT GIT_BRANCH_RESULT EQUAL 0)
        message(FATAL_ERROR "找不到git当前分支名称")
    else ()
        message("分支名称 ${GIT_BRANCH}")
    endif ()

    execute_process(
            #            COMMAND env LC_ALL=C date "+%Y-%m-%d %H:%M:%S %Z"
            COMMAND env LC_ALL=C date "+%m %b %Y %H:%M:%S %Z"
            OUTPUT_VARIABLE DATE
            OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE DATE_RESULT
    )
    if (NOT DATE_RESULT EQUAL 0)
        message(FATAL_ERROR "无法读取当前时间")
    else ()
        message("编译时间 ${DATE}")
    endif ()

    set(VERSION_COMMIT ${GIT_COMMIT})
    set(VERSION_BRANCH ${GIT_BRANCH})
    set(VERSION_BUILD_DATE ${DATE})
endif ()
```

2. CMake可以替换模板文件生成源码

模板文件如下

```c++
#ifndef VERSION_HPP
#define VERSION_HPP

// 编译版本信息
#define VERSION_MAJOR @VERSION_MAJOR@
#define VERSION_MINOR @VERSION_MINOR@
#define VERSION_MICRO @VERSION_MICRO@
#define VERSION_COMMIT "@VERSION_COMMIT@"
#define VERSION_BRANCH "@VERSION_BRANCH@"
#define VERSION_BUILD_DATE "@VERSION_BUILD_DATE@"
#define VERSION_QT "@VERSION_QT@"
#define VERSION_QT_PLATFORM "@VERSION_QT_PLATFORM@"

#endif //VERSION_HPP
```

在CMake里面相应设置值

```cmake
set(VERSION_QT ${QT_VERSION})
set(VERSION_QT_PLATFORM ${QT_PLATFORM})
```

左边是文件里面的 @@中间的名称，右边是CMake变量，然后编写替换的文件名称就可以 了

```cmake
configure_file(src/Version.hpp.in src/Version.hpp @ONLY)
```

下次执行CMake会自动在编译目录下生成.hpp文件，所以还要加上include目录才能包含头文件

```cmake
include_directories(${CMAKE_CURRENT_BINARY_DIR}/src)
```

### 2019/9/9 补充

1. CMake获取时间跨平台方法

原来的方式获取时间Windows上就会有问题，如下命令代替即可

```cmake
string(TIMESTAMP DATE "%m %b %Y %H:%M:%S")
```

时间信息就存储在`DATE`里面了

参考连接：https://cmake.org/cmake/help/v3.5/command/string.html#timestamp

2. CMake调用pkg-config

由于有些库不能通过CMake的`find_package`查找，但是可以通过pkg-config定位，比如breakpad

包的名称可以通过`pkg-config --list-all`查找

```cmake
include(FindPkgConfig)
pkg_check_modules(BREAKPAD_CLIENT REQUIRED breakpad-client)
if (${BREAKPAD_CLIENT_FOUND})
    MESSAGE(STATUS "找到breakpad-client库: ${BREAKPAD_CLIENT_INCLUDE_DIRS}")
endif()
include_directories(${BREAKPAD_CLIENT_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} ${BREAKPAD_CLIENT_LIBRARIES})
```

### 2020/2/19 补充

1. 增加msvc编译参数，增加release模式生成pdb文件

```cmake
if (CMAKE_BUILD_TYPE MATCHES Debug)
    message("Debug编译")
    set(PROJECT_BINARY_DIR ${PROJECT_SOURCE_DIR}/debug)
    add_definitions(-DDEBUG) # 加此宏是为了激活代码里面有的#ifdef DEBUG宏
else () #CMAKE_BUILD_TYPE MATCHES Release
    message("Release编译")
    set(PROJECT_BINARY_DIR ${PROJECT_SOURCE_DIR}/release)
    # 屏蔽Qt警告和调试
    add_definitions(-DQT_NO_DEBUG)
    add_definitions(-DQT_NO_DEBUG_OUTPUT)
    add_definitions(-DQT_DEPRECATED_WARNINGS)
    set(SUBSYSTEM_TYPE "WIN32")
endif ()

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR})

# 设置编译参数
if (UNIX)
    set(DEFINES "-Wunused-parameter")
    set(CMAKE_CXX_FLAGS_DEBUG "-pipe -O2 -std=gnu++11 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
    set(CMAKE_C_FLAGS_DEBUG "-pipe -O2 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
    set(CMAKE_EXE_LINKER_FLAGS_DEBUG "-fno-pie")

    set(CMAKE_CXX_FLAGS_RELEASE "-pipe -O2 -std=gnu++11 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
    set(CMAKE_C_FLAGS_RELEASE "-pipe -O2 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE "-fno-pie")
elseif (WIN32)
    if (MSVC)
        set(DEFINES_DEBUG "-D_WINDOWS -DWIN32 -D_ENABLE_EXTENDED_ALIGNED_STORAGE -DWIN64 /wd4100")
        set(CMAKE_CXX_FLAGS_DEBUG "/utf-8 /sdl /EHsc -nologo -Zc:wchar_t -FS -Zc:strictStrings -Zi -MDd -W3 -w44456 -w44457 -w44458 ${DEFINES_DEBUG}")
        set(CMAKE_C_FLAGS_DEBUG "/utf-8 /sdl -nologo -Zc:wchar_t -FS -Zc:rvalueCast -Zc:inline -Zc:strictStrings -Zc:throwingNew -Zc:referenceBinding -Zc:__cplusplus /FS -Zi -MDd -W3 -w34100 -w34189 -w44996 -w44456 -w44457 -w44458 -wd4577 -wd4467 -EHsc ${DEFINES_DEBUG}")
        set(CMAKE_EXE_LINKER_FLAGS_DEBUG "/NOLOGO /DYNAMICBASE /NXCOMPAT /INCREMENTAL:NO /DEBUG")
        # set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

        # release模式下生成pdb用于调试
        set(DEFINES_RELEASE "-D_WINDOWS -DWIN32 -D_ENABLE_EXTENDED_ALIGNED_STORAGE -DWIN64 -DQT_DEPRECATED_WARNINGS -DQT_NO_DEBUG")
        set(CMAKE_C_FLAGS_RELEASE "/utf-8 /sdl -nologo -Zc:wchar_t -FS -Zi -Zc:strictStrings -O2 -MD -W3 -w44456 -w44457 -w44458 ${DEFINES_RELEASE}")
        set(CMAKE_CXX_FLAGS_RELEASE "/utf-8 /sdl -nologo -Zc:wchar_t -FS -Zi -Zc:rvalueCast -Zc:inline -Zc:strictStrings -Zc:throwingNew -Zc:referenceBinding -Zc:__cplusplus -O2 -MD -W3 -w34100 -w34189 -w44996 -w44456 -w44457 -w44458 -wd4577 -wd4467 -EHsc ${DEFINES_RELEASE}")
        set(CMAKE_EXE_LINKER_FLAGS_RELEASE "/NOLOGO /DYNAMICBASE /NXCOMPAT /INCREMENTAL:NO /DEBUG /OPT:REF /OPT:ICF")# /NODEFAULTLIB:MSVCRT /NODEFAULTLIB:LIBC
    else ()
        message(FATAL_ERROR 不支持其他平台)
    endif ()
endif ()
```

其中`CMAKE_CXX_FLAGS_RELEASE`的`-Zi`和`CMAKE_EXE_LINKER_FLAGS_RELEASE`的`/DEBUG /OPT:REF /OPT:ICF`是生成pdb添加的

参考链接：

https://stackoverflow.com/questions/28178978/how-to-generate-pdb-files-for-release-build-with-cmake-flags

参考链接：https://cmake.org/cmake/help/v3.5/module/FindPkgConfig.html?#module:FindPkgConfig

总参考链接：

[CMake](https://cmake.org/cmake/help/latest/)

[Qt CMake](http://doc.qt.io/qt-5/cmake-manual.html)
