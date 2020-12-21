---
title: win10 使用cmd来设置环境变量
typora-copy-images-to: win10 使用cmd来设置环境变量
date: 2020-12-08 16:25:28
tags:
    - win10
    - cmd
categories: win10
---

可以使用setx命令进行设置环境变量，如果需要系统的变量这需要用管理员权限来启动CMD，否则只能设置当前用户的环境变量

```bash
setx PATH /local/share/java/bin;%PATH% # 设置当前用户的环境变量
setx /M EDITOR code # 设置系统的环境变量
```