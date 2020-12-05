---
title: pandas写入excel多个sheet
typora-copy-images-to: pandas写入excel多个sheet
date: 2020-12-05 09:42:06
tags:
    - pandas
categories: python
---
# pandas写excel文件，写入多个sheet

可以使用pandas的ExcelWriter，使用以下代码即可

```python
with pda.ExcelWriter("xx.xlsx") as writer:
    df1.to_excel(writer,sheet_name='df1')
    df2.to_excel(writer,sheet_name='df2')
```

即可将df1和df2的内容写入到xx.xlsx下的df1和df2的sheet中。