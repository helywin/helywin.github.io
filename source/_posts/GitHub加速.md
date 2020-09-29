---
title: GitHub加速
date: 2019-4-21 11:45:39
tags:
  - GitHub
---

## GitHub加速

打开<https://www.ipaddress.com/>

查询以下三个链接的DNS解析地址 

1. github.com 
2. assets-cdn.github.com 
3. github.global.ssl.fastly.net

把查到的ip地址写入host，Windows在C:\windows\system32\drivers\etc\hosts，Linux在/etc/hosts，然后刷新dns缓存

## 下载加速

1. 复制下载的url，例如
```https://github.com/googlefonts/noto-cjk/archive/NotoSansV2.001.zip```
2. 替换域名为`github-download.oss-cn-hongkong.aliyuncs.com`
```https://github-download.oss-cn-hongkong.aliyuncs.com/googlefonts/noto-cjk/archive/NotoSansV2.001.zip```
3. 复制到下载工具下载即可

### gist和代码原文件

把以下加入hosts文件

```
192.30.253.118	gist.github.com
151.101.76.133	gist.githubusercontent.com
199.232.68.133	raw.githubusercontent.com
```

