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

