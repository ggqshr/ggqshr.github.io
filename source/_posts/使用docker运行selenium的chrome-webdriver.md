---
title: 使用docker运行selenium的chrome webdriver
typora-copy-images-to: 使用docker运行selenium的chrome webdriver
date: 2021-06-08 10:35:39
tags:
    - docker
    - selenium
    - python
categories: python
---

首先使用对应的docker镜像来运行chrome webderiver，这是对应项目的主页：https://github.com/SeleniumHQ/docker-selenium
使用下面的命令来运行
```bash
$ docker run -d -p 4444:4444 -v /dev/shm:/dev/shm selenium/standalone-chrome:3.141.59-dubnium
#OR
$ docker run -d -p 4444:4444 --shm-size=2g selenium/standalone-chrome:3.141.59-dubnium
```

运行后，即可使用python运行selenium来连接，使用下面的代码即可
```python
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

driver = webdriver.Remote(
    command_executor="http://127.0.0.1:4444/wd/hub",
    desired_capabilities=DesiredCapabilities.CHROME
)

driver.get("http://httpbin.org/ip")
print(driver.page_source)
driver.close()
```
看到正常输出信息即可。

