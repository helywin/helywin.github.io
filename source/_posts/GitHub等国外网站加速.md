---
title: GitHub等国外网站加速
date: 2019-4-21 11:45:39
tags:
  - GitHub
  - Web
excerpt: 关于Github加速的问题
---

## 引言

对于程序员来说上国外的一些网站查找文档代码是一项必不可少的技能，对于这种频繁的需要一旦出现问题会耽误很多时间，比如下载或者克隆代码有时候由于下载速度需要一天，而且有时会断开连接。这篇文章也是作者自己亲身经历记录怎样应对这些问题。

## DNS污染

刷新DNS解析，更换运营商自动DNS为公共无污染DNS，比如114或者中科大的DNS

## GitHub

### 浏览和克隆代码

打开<https://www.ipaddress.com/>

查询以下三个链接的DNS解析地址 

```
github.com
assets-cdn.github.com
github.global.ssl.fastly.net
```

把查到的ip地址写入host，Windows在C:\windows\system32\drivers\etc\hosts，Linux在/etc/hosts，然后刷新dns缓存

查询结果（2020-11-21）

```
140.82.112.4 github.com
185.199.108.153/185.199.110.153/185.199.111.153 assets-cdn.github.com
199.232.69.194 github.global.ssl.fastly.net
```

### 下载加速

1. 复制下载的url，例如
   ```https://github.com/googlefonts/noto-cjk/archive/NotoSansV2.001.zip```
2. 替换域名为`github-download.oss-cn-hongkong.aliyuncs.com`
   ```https://github-download.oss-cn-hongkong.aliyuncs.com/googlefonts/noto-cjk/archive/NotoSansV2.001.zip```
3. 复制到下载工具下载即可

### gist和代码原文件

把以下加入hosts文件（2020-11-21）

```
140.82.112.4    gist.github.com
199.232.96.133    gist.githubusercontent.com
199.232.96.133    raw.githubusercontent.com
```

## StackOverflow等

很多国外网站访问慢有以下原因：

- 使用了Google的API或者字体
- 使用了recapcha
- 使用了facebook，youtube等api
- 广告跟踪脚本过多导致
- 网站本身响应慢

可以使用浏览器自带的开发者模式打开**网络**一项，重新加载网页查看各个资源加载速度和最后是否能加载成功，例如使用火狐可以禁用某些url的访问，从而可以加速同一网站网页打开速度，但这个只是用来调试的临时选项，以下浏览器插件可以增加访问速度：

- Assets CDN：加速github、gitlab访问
- Replace Google CDN：用科大镜像替换Google ajax API和字体等
- Gooreplacer：可以自定义拦截URL和重定向URL规则 ，并可以导入导出配置选项
- Ghostery：可以拦截大部分广告，跟踪器，加速网页打开速度

recapcha可以用recapcha.net的API替换