---
title: 搭建Qt开发环境
date: 2018-12-15 11:34:07
tags:
  - Qt
  - msys2
---

## 步骤

1. 安装

   进入[官网主页](http://mingw-w64.org/doku.php/start)，download里面下载msys2，或者进入镜像站点下载，比如[清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/msys2/distrib/)，下载并安装。

2. 同步镜像

   完毕后打开安装目录下的`msys2.exe`，输入`pacman -Syu`来同步最新镜像，同步完毕关闭命令行，再次打开输入`pacman -Su`，等待安装完成后关闭命令行

3. 添加国内镜像

   在安装目录下找到`etc\pacman.d`目录，下面的三个`mirrolist`文件分别改为如下内容：

   msys2

   <!-- more -->

   ```
   ##
   ## MSYS2 repository mirrorlist
   ##
   
   ## Primary
   ## msys2.org
   Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch/
   Server = http://mirrors.ustc.edu.cn/msys2/msys/$arch/
   Server = http://repo.msys2.org/msys/$arch/
   Server = https://sourceforge.net/projects/msys2/files/REPOS/MSYS2/$arch/
   Server = http://www2.futureware.at/~nickoe/msys2-mirror/msys/$arch/
   Server = https://mirror.yandex.ru/mirrors/msys2/msys/$arch/
   
   ```

   mingw32

   ```
   ##
   ## 32-bit Mingw-w64 repository mirrorlist
   ##
   
   ## Primary
   ## msys2.org
   Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686/
   Server = http://mirrors.ustc.edu.cn/msys2/mingw/i686/
   Server = http://repo.msys2.org/mingw/i686/
   Server = https://sourceforge.net/projects/msys2/files/REPOS/MINGW/i686/
   Server = http://www2.futureware.at/~nickoe/msys2-mirror/mingw/i686/
   Server = https://mirror.yandex.ru/mirrors/msys2/mingw/i686/
   ```

   mingw64

   ```
   ##
   ## 64-bit Mingw-w64 repository mirrorlist
   ##
   
   ## Primary
   ## msys2.org
   Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64/
   Server = http://mirrors.ustc.edu.cn/msys2/mingw/x86_64/
   Server = http://repo.msys2.org/mingw/x86_64/
   Server = https://sourceforge.net/projects/msys2/files/REPOS/MINGW/x86_64/
   Server = http://www2.futureware.at/~nickoe/msys2-mirror/mingw/x86_64/
   Server = https://mirror.yandex.ru/mirrors/msys2/mingw/x86_64/
   ```

   添加了清华大学和中国科技大学的镜像站点，速度更快

4. 安装Qt

   以32位Qt为例，对应上面的镜像网址为

   ```
   http://mirrors.ustc.edu.cn/msys2/mingw/i686/
   ```

   打开网址找到`mingw-w64-i686-qt5-xxx.pkg.tar.xz`，这就是Qt5的包，而`mingw-w64-i686-qt5`就是Qt5的包名

   重新打开命令行，输入`pacman -S mingw-w64-i686-qt5`就可以安装**最新**的Qt5的开发包，具体pacman命令参考[wiki](https://wiki.archlinux.org/index.php/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

   然后输入`pacman -S mingw-w64-i686-qt-creator`，安装Qt Creator

   其他开发中比较好用的工具比如clang、cppcheck、doxygen等等都可以同样的命令安装

## 环境移植

如果你需要把软件包拷贝安装到离线的电脑上安装，可以参考以下方法：

1. 拷贝文件

   需要拷贝的文件/文件夹如下：

   - `msys2-xxx.exe`，也就是一开始的安装文件
   - `var\cache\pacman`下的`pkg`文件夹（包缓存）
   - `var\lib\pacman`下的`sync`文件夹，如果执行过Syu命令则必须要（镜像同步数据）

2. 执行`pacman -Su`不联网更新本地组件
3. 在目标机器下安装msys2，然后把对应位置下拷贝过来的文件夹放到相应位置
4. 执行安装命令就可以了

## 总结

总的来说msys2的仓库集成了很多开发要用到的库和工具，对于不喜欢折腾的人来说简直是福音