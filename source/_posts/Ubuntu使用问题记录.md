---
title: Ubuntu使用问题记录
date: 2021-3-30 11:33:20
tags:
  - Ubuntu
  - Linux
excerpt: Ubuntu linux使用的一些问题记录整理
---

## 触摸屏设备自动弹出小键盘关闭

在安装了触摸屏为主屏幕的设备上，就算没打开设置里面的屏幕键盘(on-screen keyboard)，在有输入操作的情况下，比如打开终端，文本框获得输入焦点都会弹出来，后面经过搜索发现此小键盘的名字叫caribou，是gnome桌面自带的，而且要卸载可能会附带删除一些列需要的依赖，会对系统造成破坏性，后面得知gnome-extension有一个插件可以禁用

需要安装一些依赖

```bash
sudo apt install chrome-gnome-shell
sudo apt install gnome-tweak-tool gnome-tweaks
```

然后打开自带的火狐浏览器

打开网址

https://extensions.gnome.org/extension/1326/block-caribou/

对于gnome 3.36的链接还有

https://extensions.gnome.org/extension/3222/block-caribou-36/

第一次可能需要按照提示安装火狐gnome插件，装好后刷新网页，然后点击黑色的插件开关，提示确定安装，安装好就可以了

如果不生效可以重启系统或者到设置里面打开屏幕键盘选项，然后再关闭掉

## 安装微信

最新的微信用wine3.0测试安装后出现网络连接问题，于是安装的winehq最新稳定版，可以参考官方文档安装和以下配置链接

https://wiki.winehq.org/Ubuntu

https://zhuanlan.zhihu.com/p/76331687

按照这篇文章基本上可以配置好大部分功能，但是中文为□，剪贴板没用，系统图标会另外开启一个控件出来

winetricks启动可能较慢，需要耐心等待

- 中文为□

  拷贝Windows下面的字体文件到wine的相同fonts目录下，一般只需要拷贝sim开头的字体和ms开头的字体，用通配符拷贝就行了，也可以用winetricks安装cjk字体，但是需要比较久的时间

- 剪贴板没用

  使用winetricks安装ole32动态库

- 系统图标会另外开启一个控件

  安装TopIcons Plus插件

  https://extensions.gnome.org/extension/1031/topicons/

### 双系统时间

### 系统搬移

因为系统之前装在固态硬盘盒里面，由于不方便需要把系统拷贝到笔记本硬盘里面

记录下迁移的过程

1. 拷贝efi文件

   把固态盒里面的efi分区里的ubuntu目录拷贝到笔记本efi分区下面，使用的是DiskGenius

2. 拷贝数据文件

   在笔记本硬盘里分一个相同的区（由于我是直接一个分区安装的系统，多个分区可能需要分多个区或者直接把多个分区拷贝到/目录下面），然后用DiskGenius拷贝分区，使用的是拷贝文件的方式，不需要分区完全一样也可以操作，这样可以在笔记本上分个更大的分区来进行搬移

4. 设置新的UUID

   在ubuntu上用gparted给新的boot所在的分区设置新的UUID，用来区分之前的分区

3. 修改efi分区中EFI/ubuntu/grub.cfg文件

   修改配置文件使得能够找到boot分区

4. 修改/boot/grub/grub.cfg

   主要是修改一些加载参数，包括所在的磁盘位置、格式、UUID等，也可以在能够进入grub菜单后选择高级模式，用ubuntu自带的recovery tool修复grub引导

分区复杂的可能还需要修改/etc/fstab文件

附：

grub配置文件实例（带old为旧的文件，文件内容少的为efi分区中的配置文件）：

https://gist.github.com/helywin/ff10c1e9e8c0180992941a978929b604

chroot方式修复可以参考：

https://zhuanlan.zhihu.com/p/106129271