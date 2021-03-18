---
title: redis使用rdb文件进行备份
typora-copy-images-to: redis使用rdb文件进行备份
date: 2021-03-18 15:58:00
tags:
    - redis
categories: redis
---

使用redis的rdb文件来备份redis数据库的内容

首先在redis-cli中使用save后者bgsave来导出rdb数据，
然后使用`config get dir` 来查看数据导出的位置

在这个位置下找到对应的dump.rdb文件

对于另外一台redis，要导入之前，首先关闭要导入的redis，否则就算把数据复制到了指定文件夹，也会被redis关闭时的内容覆盖掉。将dump.rdb复制到其备份目录下，然后启动redis，即可完成数据的导入

在redis-cli中可以使用 `config set appendonly no`  来设置appendonly的开启与否。
其余命令的设置是相同的格式