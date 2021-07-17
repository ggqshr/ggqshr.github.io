---
title: clash在线转换服务，自己搭建
typora-copy-images-to: clash在线转换服务，自己搭建
date: 2021-07-17 19:01:10
tags:
    - clash
categories: clash
---

使用的是这个项目，https://github.com/CareyWang/sub-web
可以直接使用docker运行 
```bash
docker run -d -p 58080:80 --restart always --name subweb careywong/subweb:latest
```
打开对应网址在指定位置输入对应的vmess协议的url或者其他协议的url即可完成转换，将转换完的url复制到clash中即可使用。