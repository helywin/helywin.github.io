---
title: GitLab维护
date: 2020-07-30 09:15:47
tags:	
	- GitLab
---

## 升级

到清华大学镜像下载apt包

https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu/pool/xenial/main/g/gitlab-ce/

https://mirrors.tuna.tsinghua.edu.cn/gitlab-ee/ubuntu/pool/xenial/main/g/gitlab-ee/

这个xenial是对应Ubuntu 16.04的

直接sudo apt install ./xxxx.deb，不能暂停服务，否则无法备份然后失败

升级前需要备份

## 备份

https://docs.gitlab.com/ce/raketasks/backup_restore.html#restore-for-omnibus-installations

sudo gitlab-backup create STRATEGY=copy

同时备份 /etc/gitlab/gitlab-secrets.json，不然所有的tokens都会丢失

备份gitlab配置信息 /etc/gitlab/gitlab.rb

<!-- more -->

## 汉化

代码地址：https://gitlab.com/xhang/gitlab

下载对应的中文包，解压，删掉log，tmp文件夹

修改文件夹权限为root:root

sudo gitlab-ctl stop

sudo cp -rf 文件夹/* /opt/gitlab/embbled/services/gitlab-rails

相当于覆盖冲突的文件，注意不能直接删掉后面那个文件后移动

sudo gitlab-ctl reconfigure

sudo gitlab-ctl restart

没问题的话汉化成功

## 恢复

https://docs.gitlab.com/ce/raketasks/backup_restore.html#restore-for-omnibus-gitlab-installations

写得很详细了

## 万一玩崩了

1. 数据库没坏

sudo apt --reinstall ./xxxxx.deb 重装解决问题

2. 数据库坏了

重装并恢复

## Runner升级

https://docs.gitlab.com/runner/install/linux-manually.html

下载到/usr/local/bin并增加可执行权限

然后在gitlab后台删掉老的runner并

sudo gitlab-runner register

重新创建一个runner，保证两个runner的标签名称一致，注册为shared或者私有看原来是什么，注册完要重启gitlab服务器

sudo gitlab-ctl restart

## 502问题

Note that on a single-core server it may take up to a minute to restart Unicorn and Sidekiq. Your GitLab instance will give a 502 error until Unicorn is up again.

It is also possible to start, stop or restart individual components.

sudo gitlab-ctl restart sidekiq

Unicorn supports zero-downtime reloads. These can be triggered as follows:

sudo gitlab-ctl hup unicorn

Note that you cannot use a Unicorn reload to update the Ruby runtime.