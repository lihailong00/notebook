# playwright使用手册

[toc]



## playwright是什么

它类似selenium，可以模拟人点击web网页，我将它用于获取cookie，从而进行后续的爬虫工作。



## 安装playwright

条件：

> python3.9 
>
> pip23.0

```shell
# 安装playwright
pip install playwright
# 安装chromium，firefox等驱动（安装有点慢，我也没办法~）
python -m playwright install
```



## 简单案例

模拟百度搜索：

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # 获取浏览器
    browser = p.chromium.launch(headless=False)
    # 获取会话（不同会话的数据相互隔离）
    context = browser.new_context()
    # 获取页面
    page = context.new_page()
    # 跳转到 https://www.baidu.com
    page.goto("https://www.baidu.com")
    # 获取页面的标题
    print(page.title())
    # 定位输入框并点击
    page.locator("input[name=\"wd\"]").click()
    # 输入框中输入内容
    page.locator("input[name=\"wd\"]").fill("晓龙coding")
    # 点击搜索按钮
    page.locator("//*[@id=\"su\"]").click()

    page.close()
    context.close()
    browser.close()
```



登录教务处：

```python
```





















