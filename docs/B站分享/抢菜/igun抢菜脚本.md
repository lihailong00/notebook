# igun抢菜脚本

[toc]

## playwright是什么

它类似selenium，可以模拟人点击网页。



## 配镜像源

阿里：http://mirrors.aliyun.com/pypi/simple/

清华：https://pypi.tuna.tsinghua.edu.cn/simple/



## 安装playwright

条件：

> python3.9 
>
> pip23.0

```shell
# 安装playwright 1.32.1
pip install playwright
# 安装chromium，firefox等驱动（安装有点慢，我也没办法~）
python -m playwright install
```



## 自动生成代码

`playwright codegen`。



## 给鸽鸽点赞

```python
import asyncio

from playwright.async_api import Playwright, async_playwright, expect


async def run(playwright: Playwright) -> None:
    browser = await playwright.chromium.launch(headless=False)
    context = await browser.new_context()
    page = await context.new_page()
    await page.goto("http://104.208.106.146/")
    for i in range(100):
        await page.locator("img").nth(1).click()
        await asyncio.sleep(2)
    # ---------------------
    await context.close()
    await browser.close()


async def main() -> None:
    async with async_playwright() as playwright:
        await run(playwright)


asyncio.run(main())
```



## 模拟登录

> 人为识别验证码

```python
import asyncio

from playwright.async_api import Playwright, async_playwright, expect


async def run(playwright: Playwright) -> None:
    browser = await playwright.chromium.launch(headless=False)
    context = await browser.new_context()
    page = await context.new_page()
    await page.goto("http://104.208.106.146/")
    await page.get_by_role("link", name="登录").click()
    await page.get_by_role("group", name="用户名").get_by_role("textbox").click()
    await page.get_by_role("group", name="用户名").get_by_role("textbox").fill("123")
    await page.locator("input[type=\"password\"]").click()
    await page.locator("input[type=\"password\"]").fill("123")
    cd = input('请输入验证码: ')
    await page.get_by_role("group", name="验证码").get_by_role("textbox").click()
    await page.get_by_role("group", name="验证码").get_by_role("textbox").fill(cd)
    page.once("dialog", lambda dialog: dialog.dismiss())
    await page.get_by_role("button", name="登录").click()
    await page.get_by_role("link", name="去选菜").click()

    for i in range(100):
        page.once("dialog", lambda dialog: dialog.dismiss())
        await page.locator('//*[@id="app"]/div/div[1]/div[3]/div/div[1]/div/table/tbody/tr[17]/td[6]/div/button').click()
        await asyncio.sleep(5)

    # ---------------------
    await context.close()
    await browser.close()


async def main() -> None:
    async with async_playwright() as playwright:
        await run(playwright)


asyncio.run(main())
```





> 安装ddddocr

版本号为： 1.4.7

```python
```





