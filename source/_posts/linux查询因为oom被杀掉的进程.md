---
title: linux查询因为oom被杀掉的进程
typora-copy-images-to: linux查询因为oom被杀掉的进程
date: 2021-08-03 21:30:58
tags:
    - linux
categories: linux
---

在linux上可以使用
```bash
dmesg -T | egrep -i 'killed process'
```
来查询因为oom被杀掉的应用

[![linux查询因为oom被杀掉的进程_0](https://z3.ax1x.com/2021/08/03/fFMI1I.png)](https://imgtu.com/i/fFMI1I)

如果有被杀掉的应用，那么显示就会如上图

同时也可以使用`journalctl -k `命令来查询系统的日志，如果有的话那么会在日志中显示完整的oom被杀掉的过程。

其中`dmesg`命令是用来在Unix-like系统中显示内核的相关信息的。dmesg全称是display message (or display driver)，即显示信息。实际上，dmesg命令是从内核环形缓冲区中获取数据的。当我们在Linux上排除故障时，dmesg命令会十分方便，它能很好地帮我们鉴别硬件相关的error和warning。除此之外，dmesg命令还能打印出守护进程相关的信息，已帮助我们debug。因此OOM相关的信息可以在这里看到。

