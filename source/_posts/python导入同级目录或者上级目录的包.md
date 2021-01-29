---
title: python导入同级目录或者上级目录的包
typora-copy-images-to: python导入同级目录或者上级目录的包
date: 2021-01-29 16:59:55
tags:
    - python
    - import
categories: import
---
python导入同级目录的包或者上级目录的包的方式，比如以下的情形
```
--dir 
    | file1.py
    | dir1
        | d1.py
        | __init__.py
    | dir2
        | d2.py
        | __init__.py
    | __init__.py
```
比如d1.py需要导入dir下的file1中的内容可以使用以下的方式
```python
import sys
sys.path.append("..")
import file1
from file1 import db
```
如果是dir1下的d1需要导入d2的内容
```python
import sys
sys.path.append("..")
from dir2.d2 import xxx
```
