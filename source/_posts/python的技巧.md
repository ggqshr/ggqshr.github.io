---
title: python的技巧
date: 2020-12-01 14:14:39
tags:
    - python
categories: python
---

# python的一些小技巧

之前一直想找的一个方法，就是希望能够获得python中声明的函数的参数名，但是没有找到对应的方法，今天偶然在知乎上看到的。
可以使用Inspect模块，来获取到声明的函数的参数名

```python
import inspect

def test(a,b,c):
    print(a,b,c)

print(inspect.getfullargspec(test).args) # ['a', 'b', 'c']
```

部分转载自 https://www.zhihu.com/question/431725755/answer/1592193887