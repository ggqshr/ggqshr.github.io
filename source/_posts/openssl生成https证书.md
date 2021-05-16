---
title: openssl生成https证书
typora-copy-images-to: openssl生成https证书
date: 2021-05-16 13:39:37
tags:
    - openssl
categories: openssl
---

有些时候需要将一些服务转为https的服务，这时候就需要https证书，但是正规的https证书是需要到CA去购买的，付费不说，申请也是需要周期的，在测试时，可以使用openssl库来生成一些临时的证书

在服务端设置了自己生成的证书，在客户端只需要手动的信任这个证书。

使用下列命令来生成临时的证书
<!-- more -->
```bash
signdomain=*.webmaster.me
openssl req -nodes \
-subj "/C=CN/ST=BeiJing/L=Dongcheng/CN=$signdomain" \
-newkey rsa:4096 -keyout $signdomain.key -out $signdomain.csr
openssl x509 -req -days 3650 -in $signdomain.csr -signkey $signdomain.key -out $signdomain.crt
```

一开始定义的signdomain是要使用证书的域名，后续生成的文件都是以这个名字来进行命名的，最主要的部分应该是在

`-subj "/C=CN/ST=BeiJing/L=Dongcheng/CN=$signdomain"`中指定了我们的自定义域名，这个要和最后要使用证书的域名相对应才可以

且*.key文件是用openssl生成私钥，用来加密，而\*.csr文件是证书请求文件，包含了一些签名和对应\*.key私钥的公钥等信息，用来向CA请求证书，

而crt就是最后的证书文件，在https的握手过程中传递这个文件给客户端。而CSR文件在后续过程中一般不再使用，仅在申请时发挥作用。

另外如果自己搭建docker-registry想要使用https的话，可以按照这个博客的教程来，本文的证书生成部分就是参考[这个博客](https://www.aidmin.cn/server/docker-registry-with-self-signed-ssl-certificate.html)的内容

