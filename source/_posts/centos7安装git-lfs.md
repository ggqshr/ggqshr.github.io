---
title: centos7安装git-lfs
typora-copy-images-to: centos7安装git-lfs
date: 2021-01-30 10:12:11
tags:
	- git
	- git-lfs
categories: git
---

centos7上安装git-lfs的方式，不想win10版本的git，安装时就有lfs的支持，而是需要手动安装

首先到lfs的github上找到对应版本的rpm文件

https://github.com/git-lfs/git-lfs/releases/tag/v2.13.2

如下图位置

[![centos7安装git-lfs_0](https://s3.ax1x.com/2021/01/30/yFthWt.png)](https://imgchr.com/i/yFthWt)

点击下载，然后传到服务器上，使用命令

```bash
rpm -ivh lfs.rpm
```

即可完成安装