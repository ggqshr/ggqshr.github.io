---
title: privoxy的使用
date: 2020-12-01 19:46:57
tags:
    - privoxu
categories: 服务器环境
typora-copy-images-to: {{ title }}
---

今天新买了一个switch，但是switch链接eshop非常慢，于是想到是否可以使用代理来加速，但是尝试了一下，因为服务器上是使用的sslocal来做的代理，使用的是socks5代理，直接在wifi那里写代理的话，是无法使用的，switch这里只能使用http代理，于是搜索了一圈，发现可以使用*privoxy*来做一层包装，将socks5代理转为http，

进而了解到privoxy其实是类似与shadowsocks的一种工具，用来管理linux的全局代理，通过设置可以完成指定网站使用指定的代理的行为。

## 安装
在ubuntu和centos上都可以直接使用yum或者apt-get直接安装

```bash
yum -y install privoxy
```

## 配置
其实配置也是相对简单的，privoxy本身是一个服务，需要配置一个监听的端口，另外就是规则，哪些url需要使用代理，而那些Url不需要，privoxy的配置文件在/etc/privoxy/config

## 配置监听端口
在配置文件中搜索 `listen-address` 如下图所示：

[![privoxy的使用_0](https://s3.ax1x.com/2021/01/09/sQgMAs.png)](https://imgchr.com/i/sQgMAs)

将其改为想要使用的端口



## 配置规则

其规则的配置也比较简单，利用以下的语法即可

```
forward  .google.com  127.0.0.1:8080　
```

上述语句的意思就是*.google.com下的所有网站都需要走本地8080的代理，这是针对比较简单的http转发

如果是socks5或者socks4，则按照以下的形式：

```
forward-socks5   target_pattern  socks_proxy:port  http_proxy:port
```

前三条的意思同上，只不过第三条代表转发到那个socks5地址，最后的那个http地址是用来设置经过socks之后还需要经过哪个http代理，

基于规则的配置可以直接写到config文件中，同时也可以写到`.action`文件中，同时在config中引入：

引入的语法为:

```
actionsfile **.action
```

同时在action中的语法如下：

```
{{alias}}
direct      = +forward-override{forward .}
ssh         = +forward-override{forward-socks5 127.0.0.1:7000 .}
gae         = +forward-override{forward 127.0.0.1:8000} 
default     = direct
#==========默认代理==========
{default}
/
#==========直接连接==========
{direct} 
.edu.cn
202.117.255.
222.24.211.70
#==========SSH代理==========
{ssh}
.launchpad.net
#==========GAE代理==========
{gae}
.webupd8.org
222.24.211.70
```

通过声明别名，然后下面每个别名后对应的规则就会使用对应的代理。这样就能够配置好了

## 启动

如果通过依赖管理软件来安装，可以直接使用以下命令来启动

```bash
service privoxy start
```

# 最后

通过配置，即可达到win10上shadowsocks软件的效果，在linux下将privoxy监听的端口设置为本机的全局代理地址：

```bash
export http_proxy='http://0.0.0.0:8118' # 和config中对应
```

