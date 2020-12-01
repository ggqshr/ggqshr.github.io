---
title: 比较两个Dataframe是否相同
date: 2020-11-25 15:05:04
tags:
    - DataFrame
    - pandas
categories: python
---

经常会遇到比较两个Dataframe是否相同的问题，发现官方提供了一个方法用于比较是否相等，且能够输出不同的列

```python
from pandas.testing import assert_frame_equal

assert_frame_equal(df1,df2)
```

该方法会比对Df的value和index,如果不想比较index，也可以直接使用reset_index(drop=True)将index重置，然后在进行比较