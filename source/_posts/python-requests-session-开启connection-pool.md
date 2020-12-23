---
title: python requests session 开启connection pool
typora-copy-images-to: python requests session 开启connection pool
date: 2020-12-23 16:16:32
tags:
    - python
    - requests
categories: python
---
使用requets的session设置connection pool来提高连接的服用率，而不用一直开新的连接


核心的代码如下

```python
sess = requests.Session() # 构建 connections pool
adapter = requests.adapters.HTTPAdapter(pool_connections=20,pool_maxsize=20) # 分别自定连接池的cache的数量和最大数量
sess.mount("https://",adapter)
sess.mount("http://",adapter)
```

关于adapters.HTTPAdapter其他参数参见文档地址：
https://requests.readthedocs.io/en/master/api/?highlight=adapters.HTTPAdapter#requests.adapters.HTTPAdapter