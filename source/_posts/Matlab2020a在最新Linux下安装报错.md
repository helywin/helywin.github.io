---
title: Matlab2020a在最新Linux下安装报错
date: 2020-11-1 22:21:40
tags:
  - MATLAB
  - Linux
---

## Matlab2020a在最新Linux下安装报错

使用的是最新的Manjaro系统

错误内容: 

```
Unable to launch the MATLABWindow application
```

执行`./bin/glnxa64/MATLABWindow`可以得到具体报错的原因

首先是

```
bin/glnxa64/MATLABWindow: error while loading shared libraries: libselinux.so.1: cannot open shared object file: No such file or directory
```

安装selinux后是

```
bin/glnxa64/MATLABWindow: symbol lookup error: /usr/lib/libpango-1.0.so.0: undefined symbol: g_ptr_array_copy
```

解压安装镜像, 按照官方文档介绍修复问题

https://wiki.archlinux.org/index.php/MATLAB#Addon_manager_not_working

把解压目录里面的库文件重新链接到系统的库, 注意在`cefclient/sys/os/glnxa64`下.so和`.so.0`文件都要重新链接, 使用`ln -s /usr/lib/[名称] [当前名称]`

但是bin/glnxa64下面的一些本来是链接的解压后变成空文件了,根据执行`MATLABWindow`报错内容删除空文件重新链接, 例如

```
ln -s libicui18n.so.64.2 libicui18n.so.64
```

如果没有权限可以修改文件夹权限或者使用sudo

把所有错误修复就可以正常启动安装程序了

附带: [MATLAB安装时需要选择哪些模块](https://zhuanlan.zhihu.com/p/160659253)