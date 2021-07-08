---
title: selenium 反反爬虫
typora-copy-images-to: selenium 反反爬虫
date: 2021-07-08 22:37:10
tags:
    - python
    - selenium
categories: python
---

```python
options = webdriver.ChromeOptions()
options.add_argument("--ignore-certificate-errors")
options.add_argument("--disable-blink-features=AutomationControlled")
options.add_argument("--disable-blink-features")
options.add_experimental_option("excludeSwitches", ["enable-automation"])
options.add_experimental_option('useAutomationExtension', False)
prefs = {"profile.managed_default_content_settings.images": 2, 'disk-cache-size': 4096}
options.add_experimental_option('prefs', prefs)
root_driver = webdriver.Chrome(options=options)
root_driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {
'source': 'Object.defineProperty(navigator, "webdriver", {get: () => undefined})'
})
```

使用的测试网站`https://bot.sannysoft.com/`