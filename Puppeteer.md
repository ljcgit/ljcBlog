# Puppeteer 
## 定义
Puppeteer is a Node library which provides a high-level API to control Chrome or Chromium over the DevTools Protocol.
Puppeteer runs headless by default, but can be configured to run full (non-headless) Chrome or Chromium.

## 基本配置（基于nodejs环境）
```
# 安装命令
npm i puppeteer --save

# 错误信息
ERROR: Failed to download Chromium r515411! Set "PUPPETEER_SKIP_CHROMIUM_DOWNLOAD" env variable to skip download.

# 设置环境变量跳过下载 Chromium
set PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=1 

# 或者可以这样干，只下载模块而不build
npm i --save puppeteer --ignore-scripts

# 成功安装模块
+ puppeteer@0.13.0
added 1 package in 1.77s
```

## 常见错误说明
+ 出现(async () =>的错误说明NodeJs版本比较低，windows下可以采用去官网下载一个最新的nodeJs覆盖原来的，linux系统可以使用n命令；
+ 出现application.json不存在的错误，通过npm init命令生成application.json文件即可。

## 示例
```js
const puppeteer = require('puppeteer');

(async () => { 
    // 启动Chromium
    const browser = await puppeteer.launch({executablePath:"E:/nodejs/chrome-win/chrome.exe",ignoreHTTPSErrors: true, headless:false, args: ['--no-sandbox']});
    // 打开新页面
    const page = await browser.newPage();
	await page.goto('https://baidu.com'); 
	await page.type('#kw', 'puppeteer', {delay: 100}); //打开百度后，自动在搜索框里慢慢输入puppeteer , 
	page.click('#su') //然后点击搜索 
	await page.waitFor(1000); 
	//获取需要转向的页面
	const targetLink = await page.evaluate(() => {
		let url =  document.querySelector('.result a').href
		return url
	}); 
	console.log(targetLink); 
	await page.goto(targetLink); 
	// await page.waitFor(1000); 
	browser.close(); 
})()
```
该例子是模仿通过百度搜索puppeteer的过程。
