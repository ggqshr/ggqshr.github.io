---
title: mongodb 查询两个字段相同的行
date: 2020-11-23 11:54:27
tags:
    - mongodb
categories: mongodb相关
---

在mongodb中实现以下的sql效果，有两种实现方式
```sql
select * from db where db.a = db.b
```

## 使用$where关键字

```mongodb
db.col.find("$where":"this.field1==this.field2")
```

这个原理是使用where关键字来对每row进行一次js的运算，效率较低，当要查询的文档较多时，非常慢

## 使用aggregate

```mongodb
db.test.aggregate([
    {
        $project:{
            fields1: 1,
            fields2: 1,
            difference: { $eq: ["$fields1", "$fields2"]}
        },
    },
    {
        $match: {
        difference: true
        },
    },
]);
```

该方法是使用pipeline中的project映射出一个字段来比对两个字段