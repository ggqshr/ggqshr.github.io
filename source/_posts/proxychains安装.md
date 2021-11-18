---
title: proxychains的安装
date: 2021-11-18 17:43:00
tags:
    - proxychains
categories: linux
---
首先克隆一下对应的git repo
```bash
git clone https://github.com/rofl0r/proxychains-ng.git
```
然后即可编译安装
```bash
./configure --prefix=/usr --sysconfdir=/etc
sudo make
sudo make install
```

如果出现了`symbol lookup error: ./libproxychains4.so: undefined symbol: pthread_once`相关的错误


可以尝试将编译生成的libproxychains4.so拷贝到/usr/local/lib下面，即可解决问题