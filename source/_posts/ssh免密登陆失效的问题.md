---
title: ssh免密登陆失效的问题
typora-copy-images-to: ssh免密登陆失效的问题
date: 2021-02-28 20:37:04
tags:
    - ssh
categories: ssh
---

服务器重启后，发现原来配置过免密登陆的sftp需要输入密码，且ssh登陆也需要密码，明明配置过免密登陆，却需要输入密码，后经过确认是因为selinux的设置问题，重启后，防火墙开了起来，将其关闭即可

```bash
vim /etc/selinux/config
```
将SELINUX=enforcing 修改为 SELINUX=disabled，然后保存退出,在使用关闭防火墙
```bash
setenforce 0
```
最后重启sshd服务即可
```bash
service sshd restart 
```
