---
title: win10 git 配置ssh代理
typora-copy-images-to: win10 git 配置ssh代理
date: 2020-12-01 22:39:53
tags:
    - git
categories: git
---
配置git的ssh使用代理

同样是在 `~/.ssh/config` 下配置
## linux下

```bash
Host github.com
    User git
    Hostname github.com
    ProxyCommand nc -X 5 -x 127.0.0.1:1089 %h %p
    ServerAliveInterval 30
```

如果没有nc，则需要安装netcat

centos7 可以使用
```bash
yum -y install nc
```
但是这个版本的nc好像不能指定-X的参数
所以如果在centos7上，想要使用socks5代理的话，可以使用`connect-proxy`
可以直接使用
```bash
yum -y install connect-proxy
```

然后在.ssh/config下配置
```bash
Host github.com
    User git
    Hostname github.com
    ProxyCommand connect-proxy -S 127.0.0.1:1089 %h %p
    ServerAliveInterval 30
```

## win10下

```bash
Host github.com
    User git
    Hostname github.com
    ProxyCommand connect -S 127.0.0.1:1080 %h %p
    ServerAliveInterval 30
```

其中connect也是一个代理软件，如果没有netcat 即可使用这个命令。（没搞清是那里带的，可能是mingw64带的？）