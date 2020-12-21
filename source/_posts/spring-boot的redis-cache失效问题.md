---
title: spring boot的redis cache失效问题
typora-copy-images-to: spring boot的redis cache失效问题
date: 2020-12-08 16:12:02
tags:
    - springboot
    - java
categories: java
---

最近使用Spring-boot的cache做了系统的数据缓存，发现有时Cacheable注解不生效，调研了一下是可能是以下的情况

>1、因为@Cacheable 由AOP 实现，所以，如果该方法被其它注解切入，当缓存命中的时候，则其它注解不能正常切入并执行，@Before 也不行，当缓存没有命中的时候，其它注解可以正常工作
>
>2、@Cacheable 方法不能进行内部调用，否则缓存无法创建

发现是由于内部调用的问题，内部调用无法使用AOP，所以需要解决方法如下：

>1.不使用注解的方式，直接取 Ehcache 的 CacheManger 对象，把需要缓存的数据放到里面，类似于使用 Map，缓存的逻辑自己控制；或者可以使用redis的缓存方式去添加缓存；
>
>2.把方法A和方法B放到两个不同的类里面，例如：如果两个方法都在同一个service接口里，把方法B放到另一个service里面，这样在A方法里调B方法，就可以使用B方法的缓存。
>
>3.通过((UserService)AopContext.currentProxy().xxx()的方法获取当前类的代理类；
>
>4.通过ApplicationContext获取当前类的代理对象

最后使用的是方法4来解决的，通过Context对象来获取到上下文里生成的代理对象，即可

```java
context.getBean(Class.class).method();
```

参考：https://bobey.site/archives/456546