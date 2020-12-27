---
title: Qt MySQL库问题
date: 2019-5-7 11:49:44
tags:
  - Qt
  - MySQL
excerpt: Qt的MySQL插件和开发环境MySQL库不兼容重新编译
index_img: https://qt-web-uploads.s3.amazonaws.com/wp-content/uploads/2016/05/TheQtCompany_logo_1200x630.png
---

ubuntu 16.04环境下

使用Qt连接MySQL数据库出现如下问题：

QSqlDatabase: QMYSQL driver not loaded
QSqlDatabase: available drivers: QSQLITE QMYSQL QMYSQL3 QODBC QODBC3 QPSQL QPSQL7

使用ldd查看Qt MySQL插件的依赖库，插件位置为Qt安装目录的`plugins/sqldrivers`下面，显示如下问题：

./libqsqlmysql.so: /usr/lib/libmysqlclient.so.18: version `libmysqlclient_18' not found (required by ./libqsqlmysql.so)

在`/usr/lib/x86_64-linux-gnu`找到了下面四个文件

libmysqlclient.a  libmysqlclient.so@  libmysqlclient.so.20@  libmysqlclient.so.20.3.13

可以看出系统安装的libmysqlclient-dev库的版本和Qt编译好的动态库版本不对应，所以需要 重新编译Qt插件库并替换

在`qtbase/src/plugins/sqldrivers`目录下执行qmake和make，编译完成后到`qtbase/src/plugins/sqldrivers/plugins/sqldrivers`目录找到新的`libqsqlmysql.so`拷贝到qt安装目录`plugins/sqldrivers`下面替换即可正常使用