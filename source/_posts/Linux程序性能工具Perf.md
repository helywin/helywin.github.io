---
title: Linux程序性能工具Perf
date: 2019-5-7 11:48:24
tags:
  - Linux
  - Perf
excerpt: 有关Perf的相关介绍
---

## Linux程序性能工具Perf

有关Perf的相关介绍

https://www.ibm.com/developerworks/cn/linux/l-cn-perf1/

https://www.ibm.com/developerworks/cn/linux/l-cn-perf2/

当前只使用了一下简单的功能，后续使用会一直更新

### perf  top

执行以下命令可以查看CPU资源占用情况，细到每个函数动态库占用率

```
sudo perf top
```

查看单个进程的情况则加上参数

```
sudo perf top -p [pid]
```