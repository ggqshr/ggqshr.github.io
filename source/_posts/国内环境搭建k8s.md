---
title: Cenots7国内环境搭建k8s
typora-copy-images-to: Cenots7国内环境搭建k8s
date: 2021-05-14 16:30:21
tags:
    - k8s
categories: k8s
---

安装还是比较简单的，直接按照kubernetes官网的教程来就可以了，但是唯一比较麻烦的点在于kubernetes相关的一些镜像的下载。
这些镜像是放在gcr上的，而国内无法通过代理的方式访问gcr,只能通过替换源的方式来下载这些镜像。

该方法假设docker已经安装完毕。

# 安装对应软件

按照官方的文档，[链接](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

<!-- more -->
首先设置允许桥接流量

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

然后关闭防火墙和selinuex

```bash
systemctl stop firewalld    //关闭防火墙
systemctl disable firewalld //禁止防火墙自动启动

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

同时关闭掉交换区

```bash
swapoff -a
vi /etc/fstab
//注释掉swap分区
#/dev/mapper/centos_centos75-swap swap
//加载br_netfilter
modprobe br_netfilter
```

然后安装 kubeadm、kubelet 和 kubectl

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable kubelet
```

# 初始化

安装好上述的软件之后，开始使用kubeadm进行初始化。

首先将初始化使用的配置文件导出，修改其中的两个部分，使用如下命令导出.

```bash
kubeadm config print init-defaults > kubeadm.conf
```

然后修改kubeadm.conf中的三个地方，

```yaml
改成阿里的镜像
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers 
将1.1.1.1改成自己主机的ip
advertiseAddress: 1.1.1.1
nenetworking:
 podSubnet: 10.244.0.0/16 # 添加这个，为以后的flannel做准备
```

然后使用以下命令来拉取需要的镜像

```bash
kubeadm config images pull --config kubeadm.conf
```

如果遇到coredns/coredns下载不下来的情况，可以去dockerhub下载，然后手动tag一下，我这里在阿里的源中无法下载到coredns/coredns:1.8，所以我使用了以下的方法

```bash
docker pull coredns/coredns:1.8.0
docker tag coredns/coredns:1.8.0 registry.cn-hangzhou.aliyuncs.com/google_containers/coredns/coredns:v1.8.0 	
```

下载完成之后，即可使用以下命令来初始化

```bash
sudo kubeadm init --config kubeadm.conf 
```

最后看到如下的页面，就代表配置成功了

[![国内环境搭建k8s_0](https://z3.ax1x.com/2021/05/16/gcR2sP.png)](https://imgtu.com/i/gcR2sP)

然后设置网络相关，使用以下命令

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

最后在master节点上使用`kubectl get nodes` 看到master节点的状态是ready就可以了,如果不行的话执行一下输出中的如下部分，将配置文件复制并改一下权限

[![国内环境搭建k8s_1](https://z3.ax1x.com/2021/05/21/gH581J.png)](https://imgtu.com/i/gH581J)

在其他机器上只需要使用其输出的命令即可加入到主节点当中

```bash
kubeadm join 主机ip:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:fcccd18a50172222ac630af392f2b196da4690c70b2298e18657e30105933
```

如果只想搭建一个单节点的k8s，想在master上面运行pods，则需要以下命令

```bash
kubectl taint node localhost.localdomain node-role.kubernetes.io/master-
```

如果在运行pod时，使用describe 发现pods有如下日志：

> Flannel (NetworkPlugin cni) error: /run/flannel/subnet.env: no such file or directory

可以参考这个的解决方案  https://github.com/kubernetes/kubernetes/issues/70202#issuecomment-481173403

另外也可以参考[这种方式](https://segmentfault.com/a/1190000022369750)，自己搭建gcr的代理镜像服务器，但是我在搭建k8s的1.21.0版本时，发现有些镜像使用这种方式下载不到


## 其他问题
如果自己安装的k8s 是无法使用LoadBalance类型的service的，需要自己指定一个外部的ip，如果不指定的话，
使用`kubectl get svc` 查看对应LoadBalancer的外部IP一直是pending
在yml文件中指定 externalIPs字段
```yaml
...
spec:
  type: LoadBalancer
  externalIPs:
  - 192.168.0.10
```

[![国内环境搭建k8s_2](https://z3.ax1x.com/2021/05/21/gHIgxJ.png)](https://imgtu.com/i/gHIgxJ)

参考链接 https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending