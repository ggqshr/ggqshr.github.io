---
title: bash脚本的一些设置
typora-copy-images-to: bash脚本的一些设置
date: 2021-12-07 17:27:47
tags:
    - bash
categories: 脚本相关
---
在编写bash脚本时有一些通用的起手式

```bash
#!/usr/bin/env bash

#当使用了未初始化的变量时，设置set -u 或 set -o nounset，可以让程序强制退出
set -u
# set -o errexit 或set -e 一旦有任何一个语句返回非0值，则退出bash 
set -e
#将每行执行的命令输出，set -o xtrace或set -x
set -x
```
