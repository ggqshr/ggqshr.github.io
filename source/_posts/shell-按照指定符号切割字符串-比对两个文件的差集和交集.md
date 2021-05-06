---
title: shell 按照指定符号切割字符串 比对两个文件的差集和交集
typora-copy-images-to: shell 按照指定符号切割字符串 比对两个文件的差集和交集
date: 2021-05-06 09:30:27
tags:
    - shell命令
categories: shell命令
---

使用xargs可以按照指定符号切割字符串
```bash
head -1 file.csv | xargs -d ',' -n echo 
```

使用grep可以方便的取两个文件的交集和差集，但是要求每行都是一个item
```bash
grep -vFf file1 file2  # 取出现在file2中而没出现在file1中的行，即file2-file1
grep -Ff file1 file2 # 取file1和file2的交集
```

原理是使用grep的查找功能，`grep -Ff file1 file2` 的含义是在file2中查找file1中的内容
而`grep -vFf file1 file2` 中 -v的含义是反向匹配，则是查找出现在file2中而未出现在file1中的行


