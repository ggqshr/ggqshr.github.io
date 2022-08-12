---
title: selenium反爬
typora-copy-images-to: selenium反爬
date: 2022-08-12 20:59:30
tags:
    - python
    - selenium
    - 爬虫
    - 反爬
categories: 爬虫
---

参考<https://cloud.tencent.com/developer/article/1786102>

使用extract-stealth-evasions，github地址：https://github.com/berstend/puppeteer-extra/tree/master/packages/extract-stealth-evasions
需要安装node环境，直接使用npm
```bash
npx extract-stealth-evasions
```

生成`stealth.min.js`

然后在selenium启动时，将其作为自定义脚本执行，加入以下的代码

```python
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument("--disable-blink-features=AutomationControlled")
driver = webdriver.Chrome(options=chrome_options)
with open("stealth.min.js") as f:
    js = f.read()
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    "source": js
})
```

如果是使用的remote的chromedriver,可以使用如下代码，参考<https://github.com/SeleniumHQ/selenium/issues/8672#issuecomment-699676869>
```python
def send(driver, cmd, params={}):
    resource = "/session/%s/chromium/send_command_and_get_result" % driver.session_id
    url = driver.command_executor._url + resource
    body = json.dumps({'cmd': cmd, 'params': params})
    response = driver.command_executor._request('POST', url, body)
    print(response)
    return response.get('value')

def add_script(driver, script):
    send(driver, "Page.addScriptToEvaluateOnNewDocument", {"source": script})

add_script(self.root_driver,js)
```