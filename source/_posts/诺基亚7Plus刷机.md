---
title: 诺基亚7Plus刷机
author: helywin
date: 2020-4-10 10:23:45
category: 刷机
excerpt: 诺基亚7Plus刷机
---

# 诺基亚7Plus刷机



已解锁

允许usb调试，输入以下命令进入Download mode

```shell
adb reboot fastboot
```

准备好twrp镜像，[下载地址](https://sourceforge.net/projects/b2n-sprout/files/TWRP-TEN/POB/)

由于使用A/B分区，在当前为A分区的时候刷机会刷到B分区，采用临时引导recovery的方式切换分区和刷机

输入以下命令进入临时recovery系统

```shell
fastboot boot D:\刷机\twrp-3.4.0-0-B2N_sprout-OOB-10.img
```

进入recovery等待加载完毕，然后拷贝下载好的rom到手机，[下载地址](https://sourceforge.net/projects/b2n-sprout/files/LineageOS/)(我用的是LineageOS，名字叫lineage-18.1-20210330-UNOFFICIAL-Onyx.zip)

在重启选项里面查看当前分区，比如当前是B，刷机会把系统刷到A分区

所以需要重启更改启动分区，在`重启选项`中选择`分区A`(因为我的系统在B分区)

这个信息在刷机时也提示

```
Flashing A/B zip into inactive slot:B
```

刷机前记得清除

记得刷完后重启到刷好机的分区就行了