---
title: git-lfs 不下载文件
typora-copy-images-to: git-lfs 不下载文件
date: 2021-02-27 19:18:36
tags:
    - git
    - git-lfs
categories: git
---
https://sabicalija.github.io/git-lfs-intro/
https://zzz.buzz/zh/2016/04/19/the-guide-to-git-lfs/

本地可以使用
```bash
git lfs install --local --skip-smudge 
```

全局就是
```bash
git lfs install --skip-smudge
```

下载某个文件是
```bash
git lfs pull --include="filename"
```