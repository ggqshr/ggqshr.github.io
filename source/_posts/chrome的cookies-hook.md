---
title: chrome的cookies hook
typora-copy-images-to: chrome的cookies hook
date: 2021-12-26 22:01:15
tags:
    - 反爬
    - 爬虫
categories: 爬虫
---
# 前言

相关代码进攻技术交流使用，禁止用于任何商业盈利用途，该代码造成的后果和本人无关。

# 背景

在反爬时，尝尝会遇到一些cookies相关的反爬，如果直接使用请求工具请求的话，是会被拦截住，导致无法爬取内容，因此需要js的环境来执行这些代码，设置正确的cookies从而能够访问到网站。

# 方法

使用chrome的油猴插件（直接搜索油猴插件安装即可），自定义一个cookies的hook函数，在有代码设置cooklies值的时候进行debug，从函数的调用栈中即可得到该cookies的生成流程，然后反向破解，即可得到cookies的生成方法。

下面就是这段cookies hook的代码，

```javascript
(function () {
    'use strict';
    var cookie_cache = document.cookie;
    Object.defineProperty(document, 'cookie', {
        get: function() {
            console.log('getcookie');
            debugger;
            return "";
        },
        set: function(value) {
            console.log('setcookie', value);
            if (value.indexOf('想要破解的cookies名') != -1) { // 修改名字
               debugger;
            }
            return value;
        },
    });
})();
```

在油猴中新建一个用户脚本，然后将其复制到脚本中，在目标网站清空所有的cookies，然后打开开发者工具，刷新页面，等待页面刷新，则当代码在设置我们制定的值时就会进入debug状态，这时我们可以通过函数的调用栈，一层层的找，从而找到cookies的生成方式
