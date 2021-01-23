---
title: pandas更新多列
typora-copy-images-to: pandas更新多列
date: 2021-01-23 16:26:34
tags:
    - pandas
categories: python
---
pandas同时更新多列满足特定条件的值
```python
data.loc[data.field == condition,col1:col2] = value
```
这里col1:col2需要是连续的多列