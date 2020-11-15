---
title: 设置git的特定网站使用代理
date: 2020-11-15 20:03:26
tags:
 - git
categories: git
---
# 设置git特定网站使用代理

## 1.命令行的方式


直接使用命令即可设置特定网站的使用特定代理，比如github


```bash
git config --global http.https://github.com.proxy http://127.0.0.1:1080

# 取消代理
git config --global --unset http.https://github.com.proxy
```

如果不想设置全局的，则可以设置repository特定的代理，如下

```bash
git config --local http.https://github.com.proxy http://127.0.0.1:1080

# 取消代理
git config --local --unset http.https://github.com.proxy
```

## 2. 编辑配置文件的方式

直接编辑git的配置文件，该文件的位置为`~/.gitconfig`

```
[http "https://github.com"] # 添加特定网站
	proxy = http://127.0.0.1:1080
    
[http] # 设置全部http都使用某个代理
    proxy = http://127.0.0.1:1080 
```