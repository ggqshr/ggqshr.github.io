---
title: ubuntu上使用v2ray和ss-tproxy做透明代理
typora-copy-images-to: ubuntu上使用v2ray和ss-tproxy做透明代理
date: 2021-12-09 15:04:10
tags:
    - v2ray
    - linux 
categories: linux
---
使用v2ray与ss-tproxy来做透明代理，代理本机的所有流量。
ss-tproxy项目链接：https://github.com/zfl9/ss-tproxy

## 安装v2ray

透明代理使用的v2ray不能使用docker安装，使用这个[项目](https://github.com/v2fly/fhs-install-v2ray)提供的脚本来安装v2ray。

<!-- more -->

```bash
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

脚本会将配置文件安装到`/usr/local/etc/v2ray/config.json`。然后根据自己的v2ray服务器地址编写配置文件，配置文件基本上没有什么变化，只需要在原来的`inbounds`中新增一个类型为`dokodemo-door`的入口即可，且`ss-tproxy`不建议在v2ray中做分流，则outbound中也只需要有一个出口即可，则整个配置文件的看起来像下面这样（其中出口处的信息已经隐去）

```json

{
    "inbounds": [
        {
            "tag": "http", // 一个http入口可以用来做测试，测试出口是否通到服务器
            "port": 55555,
            "listen": "0.0.0.0",
            "protocol": "http",
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            },
            "settings": {
                "udp": false,
                "allowTransparent": false
            }
        },
        {
            "tag": "in_transparent", // 用来做透明代理的入口
            "port": 55556,
            "listen": "0.0.0.0",
            "protocol": "dokodemo-door",
            "settings": {
                "network": "tcp,udp",
                "followRedirect": true
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            },
            "streamSettings": {
                "sockopt": {
                    "tproxy": "redirect"
                }
            }
        }
    ],
    "outbounds": [
        {
            "tag": "proxy", // 填写自己服务器的配置
            "protocol": "vmess",
            "settings": {
            },
        }
    ]
}

```

然后即可使用命令来启动v2ray

```bash
systemctl start v2ray.service
```

启动后，通过命令查看程序状态

```bash
systemctl status v2ray.service
```

可以看到程序状态为`active`，说明正常启动。

## 安装ss-tproxy以及其相关依赖

根据官方提供的相关依赖安装，根据自己的系统安装下列依赖。

<https://github.com/zfl9/ss-tproxy/wiki/Linux-%E9%80%8F%E6%98%8E%E4%BB%A3%E7%90%86#%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96>

装好了依赖之后，安装ss-tproxy，参考：<https://github.com/zfl9/ss-tproxy#%E5%AE%89%E8%A3%85%E8%84%9A%E6%9C%AC>

```bash
git clone https://github.com/zfl9/ss-tproxy
cd ss-tproxy
chmod +x ss-tproxy

install ss-tproxy /usr/local/bin
install -d /etc/ss-tproxy
install -m 644 ss-tproxy.conf gfwlist* chnroute* ignlist* /etc/ss-tproxy
install -m 644 ss-tproxy.service /etc/systemd/system # 可选，安装 service 文件
```

然后根据我们的设置修改`ss-tproxy`的配置文件

```bash
vim /etc/ss-tproxy/ss-tproxy.conf
```

然后按照我们`v2ray`的配置，修改配置文件，下面只列出了需要修改的部分

```conf
## proxy
proxy_svraddr4=(1.1.1.1)) # v2ray服务器的 IPv4 地址或域名，允许填写多个服务器地址，空格隔开
proxy_svrport='80,443'   # v2ray服务器的监听端口，可填多个端口，格式同 ipts_proxy_dst_port
proxy_tcpport='55556'    # ss/ssr/v2ray 等本机进程的 TCP 监听端口，该端口支持透明代理，对应v2ray设置中协议为dokodemo-door的端口
proxy_udpport='55556'    # ss/ssr/v2ray 等本机进程的 UDP 监听端口，该端口支持透明代理，对应v2ray设置中协议为dokodemo-door的端口
proxy_startcmd='systemctl start v2ray.service'     # 用于启动本机代理进程的 shell 命令，该命令应该能立即执行完毕
proxy_stopcmd='systemctl stop v2ray.service'      # 用于关闭本机代理进程的 shell 命令，该命令应该能立即执行完毕
```

修改完毕后，启动`ss-tproxy`。
```bash
sudo ss-tproxy start
```

然后查看状态

```bash
sudo ss-tproxy status
```

如果显示都是running则说明没有问题。

然后可以测试一下全局透明代理是否生效，首先测试一下国内的百度能够访问

```bash
curl -4vsSkL https://www.baidu.com
```

如果启动dnsmasq失败，需要关闭ubuntu的默认dns，可以参考`https://www.netroby.com/view/4076`

如果能看到网页源码的输出，则说明没有问题

然后来测试国外网址的代理是否生效，新开一个ssh连接到服务器，使用`journalctl -u v2ray -f` 来查看v2ray的输出。

然后在原来的ssh中执行以下命令。

```bash
curl -4vsSkL https://www.google.com
```

如果能看到网页源码输出，且在v2ray的日志中能够看到类似下面的日志，就说明国外的流量被正确的分流到了v2ray中，透明代理生效了.

```
Dec 09 07:39:52 dev v2ray[57363]: 2021/12/09 07:39:52 [Info] [326836364] proxy/dokodemo: received request for 172.31.197.148:50420
```

## 其他

至此，v2ray和ss-tproxy都安装完成，实现了机器上的透明代理，机器上所有的流量都会先经过`ss-tproxy`来分流，如果是访问国外的流量会被分流到v2ray中做代理。

如果想要关闭`ss-tproxy`，使用下列命令
```bash
ss-tproxy stop
```