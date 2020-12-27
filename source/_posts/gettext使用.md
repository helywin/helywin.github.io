---
title: gettext使用
date: 2020-8-14 14:13:52
tags:
  - gettext
excerpt: 介绍了怎样使用gettext实现简单的程序动态翻译
---

## 简介

GNU getttext是实现软件国际化的一套多语言工具，运行程序运行时根据不同的语言环境切换不同的程序语言，对应Qt的Linguist

## 获取库

使用vcpkg一键安装

## 翻译

### 程序代码

示例程序代码如下

```c++
#include <iostream>
#include <libintl.h>

#define PACKAGE "test_gettext"

int main()
{
    setlocale(LC_ALL,"");    //设置当前语言为系统默认值
    bindtextdomain(PACKAGE, "lang");    //设置翻译文件的路径
    textdomain(PACKAGE);    //设置当前翻译文件的名称
    std::cout << gettext("TEXT_HELLO") << std::endl;
    return 0;
}
```

经过上述设置后翻译文件应当放在程序目录的`lang/<Locale>/LC_MESSAGES/test_gettext.mo`

其中`Locale`后续配置时会提到

### 生成模板

```shell
xgettext --package-name test_gettext --package-version 1.0 --default-domain test_gettext test_gettext.cpp -o test_gettext.pot --from-code=UTF-8
```

生成模板文件`test_gettext.pot`

<!-- more -->

```ini
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER
# This file is distributed under the same license as the test_gettext package.
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
#, fuzzy
msgid ""
msgstr ""
"Project-Id-Version: test_gettext 1.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-08-14 14:24+0800\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"Language: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=CHARSET\n"
"Content-Transfer-Encoding: 8bit\n"

#: test_gettext.cpp:11
msgid "TEXT_HELLO"
msgstr ""
```

### 生成翻译文本

```shell
msginit --no-translator --locale zh_CN -o test_gettext.po -i test_gettext.pot
```

生成文件`test_gettext.po`

```ini
# Chinese translations for test_gettext package.
# Copyright (C) 2020 THE test_gettext'S COPYRIGHT HOLDER
# This file is distributed under the same license as the test_gettext package.
# Automatically generated, 2020.
#
msgid ""
msgstr ""
"Project-Id-Version: test_gettext 1.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-08-14 14:30+0800\n"
"PO-Revision-Date: 2020-08-14 14:30+0800\n"
"Last-Translator: Automatically generated\n"
"Language-Team: none\n"
"Language: zh_CN\n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=ASCII\n"
"Content-Transfer-Encoding: 8bit\n"

#: test_gettext.cpp:11
msgid "TEXT_HELLO"
msgstr ""
```

把ASCII换成UTF-8，加上翻译`你好，世界`

`zh_CN`参数决定了该翻译应该放在`lang/zh_CN/LC_MESSAGES/test_gettext.mo`目录下被读到

### 生成翻译文件

```shell
msgfmt --check --verbose -o test_gettext.mo test_gettext.po
```

最终就能得到`test_gettext.mo`了

### 运行

指定环境变量`LANG`

```shell
set LANG=zh_CN.UTF-8
.\test_gettext.exe
```

### 另外

网上有一些po文件编辑器，在翻译时可以显示对应代码的位置方便对应起来，稍微大一点的内部项目可以采用像Weblate这种平台多人协助翻译，开源项目同样有在线翻译平台，可以参考一些大型开源项目的作法
