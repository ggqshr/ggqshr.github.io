---
title: prometheus-operator扫描Podmonitor以及ServiceMonitor的设置.md
date: 2021-08-23 11:29:00
tags:
    - k8s
    - prometheus-operator
    - prometheus
categories: k8s
---
参考了博客：https://www.jianshu.com/p/bd7ba22cf52b
其中说到了Prometheus-operator在扫描PodMonitor以及ServiceMonitor的配置是通过monitoring.coreos.com/v1中的Prometheus来定义的，通过serviceMonitorSelector和podMonitorSelector决定哪些ServiceMonitor和PodMonitor生效。如果选择器为空({})意味着会选择所有的对象。
<!-- more -->
则如果安装Prometheus-operator之后，且设置了对应的PodMonitor，但是还是没有看到对应的服务被发现，则可以查看一下Prometheus资源的设置，看下是不是需要给PodMonitor以及ServiceMonitor打上什么特殊的标签才能够被发现，如果不想打标签，可以直接修改资源的设置。
使用以下命令来修改设置
```bash
kubectl edit prometheuses.monitoring.coreos.com -n monitoring name
```
搜索podMonitorSelector，将设置修改为想要的配置，然后保存退出即可生效。