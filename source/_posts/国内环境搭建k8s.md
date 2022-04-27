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

# 1. 安装k8s的runtime

这里选择使用docker作为k8s的runtime，可以直接使用docker官方提供的一键脚本
```bash
wget -qO- https://get.docker.com/ | sh
```

然后启动一下docker
```bash
systemctl start docker
```

查看一下docker的状态

```bash
systemctl status docker
```

设置docker使用的cgroupdriver，修改成和kubelet的一致，使用以下命令
```bash
cat <<EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
```

如果不修改这个cgroupdriver在`kubelet`启动时会出现如下报错
```
"Failed to run kubelet" err="failed to run Kubelet: misconfiguration: kubelet cgroup driver: \"systemd\" is different from docker cgroup driver: \"cgroupfs\""
```

然后运行docker即可
```bash
systemctl restart docker
```

然后查看docker的运行情况
```bash
systemctl status docker
```

看到以下的输出就说明docker正常启动了
[![国内环境搭建k8s_0](https://s1.ax1x.com/2022/04/22/L25HTx.png)](https://imgtu.com/i/L25HTx)

# 2. 安装kubelet、kubeadm、kubectl

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

然后关闭防火墙和selinuex，如果是公网服务器不建议关闭防火墙

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
//注释掉swap分区,如果有的话
#/dev/mapper/centos_centos75-swap swap
//然后保存退出，执行命令加载br_netfilter
modprobe br_netfilter
```

然后安装 kubeadm、kubelet 和 kubectl，这里选择安装版本为`1.22.9`

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

然后可以使用以下命令查看有哪些版本，本文已经选定了版本，可以不运行下面的命令，如果需要安装其他版本的话，可以运行一下下方的命令看有哪些版本可选
```bash
yum list kubelet kubeadm kubectl  --showduplicates|sort -r
```

然后安装对应的版本

```bash
yum install kubelet-1.22.9-0 kubeadm-1.22.9-0 kubectl-1.22.9-0

systemctl enable kubelet
```

**注意**：若为worker节点，只需要进行到此步骤即可，若为master节点还需继续往下执行
worker节点在安装了kubelet、kubeadm已经kubectl之后就可以被初始化了，等到master安装完成使用`kubeadm join`即可加入到集群中

# 3. 初始化

安装好上述的软件之后，开始使用kubeadm进行初始化。

首先将初始化使用的配置文件导出，修改其中的两个部分，使用如下命令导出.

```bash
kubeadm config print init-defaults > kubeadm.conf
```

然后修改kubeadm.conf中的三个地方，首先将镜像源改成阿里的镜像

```yaml
...
advertiseAddress: 10.0.0.2 # apiserver绑定的ip，一般写内网的ip地址
...
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers 
...
networking:
 podSubnet: 10.244.0.0/16 # 添加这个，为以后的flannel做准备
```

然后使用以下命令来拉取需要的镜像

```bash
kubeadm config images pull --config kubeadm.conf
```

正常来讲，可以看到以下的输出
[![国内环境搭建k8s_1](https://s1.ax1x.com/2022/04/22/L25qk6.png)](https://imgtu.com/i/L25qk6)

下载完成之后，即可使用以下命令来初始化

```bash
sudo kubeadm init --config kubeadm.conf 
```

最后看到如下的页面，就代表配置成功了

[![国内环境搭建k8s_0](https://z3.ax1x.com/2021/05/16/gcR2sP.png)](https://imgtu.com/i/gcR2sP)

然后按照其提示，配置一下kubectl需要的配置文件
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

然后设置网络相关，使用以下命令

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

最后在master节点上使用`kubectl get nodes` 看到master节点的状态是ready就可以了,如果不行的话执行一下输出中的如下部分，将配置文件复制并改一下权限

[![国内环境搭建k8s_1](https://z3.ax1x.com/2021/05/21/gH581J.png)](https://imgtu.com/i/gH581J)

在其他worker节点上只需要使用其输出的命令即可加入到主节点当中

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


## 3.1. 其他问题
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

如果下载不了镜像也可以采取这种方式，自己通过阿里云的服务来构建https://mp.weixin.qq.com/s/kf0SrktAze3bT7LcIveDYw

## 3.2. kubeadm 初始化失败
有些时候kubeadm可能会初始化失败，这时候需要将一些配置重置，使用以下命令
```bash
kubeadm reset
```