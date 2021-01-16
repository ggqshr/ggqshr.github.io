---
title: python 正则匹配重复的字符
typora-copy-images-to: python 正则匹配重复的字符
date: 2021-01-16 16:42:12
tags:
    - python
    - 正则
categories: python
---
使用python的正则来匹配连续的重复字符，比如
> 111111111
>
> aaaaaaaaaaaaaaaaaaaaaaa
>
> bbbbbbbbbbb
>
> 哈哈哈哈哈哈哈

可以使用以下的代码来匹配

```python
re.search("(.)\1{5,}")
```

`(.)`这就是匹配任何字符，`\1`是引用第一个分组，就是`(.)`，`{5,}`的含义是一个字符至少重复五次，但是加上`\1`的引用，则整体的含义是匹配至少出现6次的重复字符

```python
>>> re.search(r"(.)\1{5,}",'aaaaaa') # 6个
<re.Match object; span=(0, 6), match='aaaaaa'>
>>> re.search(r"(.)\1{5,}",'aaaaa')
>>>
```



