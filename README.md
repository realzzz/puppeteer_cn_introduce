

##### Puppeteer的[GitHub](https://github.com/GoogleChrome/puppeteer)链接 ``本文是对该链接的翻译，扩充解释和举例说明``

> Puppeteer 是谷歌公司最近推出的基于Node开发的一套高级API库，通过[开发协议](https://chromedevtools.github.io/devtools-protocol/)来控制[无界面](https://developers.google.com/web/updates/2017/04/headless-chrome)的浏览器。

> 通俗的说就是有这么一套API, 可以用来控制浏览器的行为，比如打开网页，查看控件／文本，填入文字，点击按钮，滑动屏幕. 而这个浏览器可以是不显示界面的 (这对于在命令行情况下的运行非常方便)

###### 我能用这套API干嘛呢?

几乎所有你能在浏览器上做的事情, 通过调用puppeteer API 也能够实现， 比如：

*   生成浏览器页面的屏幕截图或者是pdf文件
*   方便的抓取[单页面](https://en.wikipedia.org/wiki/Single-page_application)和预渲染页面的信息内容
*   网站爬虫
*   自动化执行页面提交，UI自动化测试，键盘输入等.
*   可以建立基于最新Chrome和Javascript的测试环境
*   抓取并跟踪网站的执行时间轴，帮助分析效率问题

## 入门指南

### 安装

> *注意: Puppeteer最低要求Node版本v6.4.0, 但是下面的例子中出现的 [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)/[await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) 的用法最低要求Node版本v7.6.0

在node工程中使用Puppeteer, (添加依赖库) 执行 :
```
yarn add puppeteer
# or "npm i puppeteer"
```


> **注意*: 安装Puppeteer的时候会安装最新版本的Chromium ( 预估大小 ~71Mb Mac, ~90Mb Linux, ~110Mb Win) 来确保API可以执行. 如果你确定不需要下载，请设置 [环境变量](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#environment-variables).


### 使用

Puppeteer 和其他驱动浏览器来执行测试的框架([Casperjs](http://casperjs.org/s), [Selenium](http://www.seleniumhq.org/) ``笔者: 以后会分别写文章介绍``)类似. 你创建一个浏览器的实例，打开页面，通过[Puppeteer's API](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#) 操作页面上的元素.

**举例** - 访问 https://example.com 并保存页面截图为 *example.png*:
```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch(); // 创建浏览器实例
  const page = await browser.newPage(); // 创建新的浏览器页面
  await page.goto('https://example.com'); //  页面访问地址 http://example.com
  await page.screenshot({path: 'example.png'}); // 页面截图 example.png

  await browser.close(); // 关闭浏览器
})();
```


Puppeteer 设置浏览器页面为 800像素 x 600像素, 屏幕截图也依据这个大小. 如你需要调整页面大小，可以通过  [`Page.setViewport()`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetviewportviewport).

**举例** - 创建PDF.

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://news.ycombinator.com', {waitUntil: 'networkidle2'});
  await page.pdf({path: 'hn.pdf', format: 'A4'});

  await browser.close();
})();
```
上例中waitUntil表示等待的时长，参数定义在这里[waitUntil](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#) - 搜索*waitUntil*
[`Page.pdf()`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagepdfoptions) 访问这里有更多关于创建PDF的信息.

**举例** - 通过页面上下文 (*context*) 获取页面信息

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');

  // Get the "viewport" of the page, as reported by the page.
  const dimensions = await page.evaluate(() => { // 通过evaluate执行页面js
    return {
      width: document.documentElement.clientWidth, // 页面宽度
      height: document.documentElement.clientHeight, // 页面高度
      deviceScaleFactor: window.devicePixelRatio // 设备像素比
    };
  });

  console.log('Dimensions:', dimensions);

  await browser.close();
})();
```
访问 [`Page.evaluate()`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevaluatepagefunction-args) 获得更多关于 `evaluate` 和相关功能例如 `evaluateOnNewDocument` and `exposeFunction`的介绍。

## 默认的运行设置

**1. 使用无界面模式**

Puppeteer 运行 Chromium in [无界面模式](https://developers.google.com/web/updates/2017/04/headless-chrome). 如果你需要运行一个完整的 Chromium, 在创建浏览器的时候请设置 ['headless' ](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions) 选项

```js
const browser = await puppeteer.launch({headless: false}); // 默认为true
```
**2. 自定义运行的 Chromium**

默认情况下, Puppeteer 会选择自行选择下载 Chromium 来确保其API 在当前环境下正常运行. 如果确认需要运行不同版本的 Chromium, 在创建浏览器的时候传入executablePath参数，值为目标浏览器的可执行路径:

```js
const browser = await puppeteer.launch({executablePath: '/path/to/Chrome'});
```
**3. 创建新的用户信息**

Puppeteer 会自动创建Chromium 用户信息，同时 **每次执行完都清除**.

## API 文档

完整 [API 文档](docs/api.md) 和 [例子](https://github.com/GoogleChrome/puppeteer/tree/master/examples/).

## 调试技巧

1. 显示界面 - 最直观的调试方法就是看到界面上发生了什么. 通过创建完整浏览器来实现，选项 `headless: false`:

    ```js
    const browser = await puppeteer.launch({headless: false});
    ```

1. 让执行慢下来 - `slowMo` 选项 可以指定毫秒值，让 Puppeteer 的执行慢下来 ，也对调试有帮助

    ```js
    const browser = await puppeteer.launch({
      headless: false,
      slowMo: 250 // slow down by 250ms
    });
    ```

1. 获取Console的输出 - 你既可以监听 `console` 事件, 也可以通过 `page.evaluate()`来打印。

    ```js
    page.on('console', msg => console.log('PAGE LOG:', ...msg.args));

    await page.evaluate(() => console.log(`url is ${location.href}`));
    ```

1. 启用详细日志 - 所有API调用和内部协议交互都会被记录在 `puppeteer` 名字空间的 [`debug`](https://github.com/visionmedia/debug) 模式下.

   ```sh
   # 所有详细的日志
   env DEBUG="puppeteer:*" node script.js

   # 通过名字空间来控制调试日志的输出
   env DEBUG="puppeteer:*,-puppeteer:protocol" node script.js # 除了protocol外的所有消息
   env DEBUG="puppeteer:session" node script.js # 只需要protocol session 消息
   env DEBUG="puppeteer:mouse,puppeteer:keyboard" node script.js # 只输出鼠标和键盘日志

   # Protocol 的交互消息会很多. 这里的例子说明了如何过滤掉所有Netwok的消息。
   env DEBUG="puppeteer:*" env DEBUG_COLORS=true node script.js 2>&1 | grep -v '"Network'
   ```

### 额外的例子

这些例子从 [Issue](https://github.com/GoogleChrome/puppeteer/issues) 页面归纳而来，如果有额外的需要请留言。


1. 如何模拟页面点击？

    通过以下page的接口, 相关 [issue](https://github.com/GoogleChrome/puppeteer/issues/40)
   ```js
   page.mouseMoved(x, y, options = {})
   page.mousePressed(x, y, options = {})
   page.mouseReleased(x, y, options = {})
   page.tap(x, y, options = {})
   page.touchmove()
   page.touchend()
   ```
1. 如何上下翻动页面？

    通过调用page.evaluate中的 window.scrollBy来实现, 相关 [issue](https://github.com/GoogleChrome/puppeteer/issues/305)
    ```js
    page.evaluate(_ => { window.scrollBy(0, window.innerHeight);
    ```
1.  避免页面ssl认证错误信息
     通过puppeteer option [ignoredHTTPErrors](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions) 实现
1. page.evalute 能否返回page DOM?
      你可以传入 ObjectHandle到page.evaluate中成为DOM元素，但当DOM被返回的时候则成  为对应的 ObjectHandle. [issue](https://github.com/GoogleChrome/puppeteer/issues/382)
      如果需要返回，也可以返回实际需要的值，例如：
    ```js
    const list = await page.evaluateHandle(() => {
    return Array.from(document.getElementsByTagName('a')).map(a => a.href);
    });
    console.log(await list.jsonValue());
    ```
    相关 [iusse](https://github.com/GoogleChrome/puppeteer/issues/1263)
1. 如何读取和设置cookies?
    通过page.setCookie 和 page.cookies 接口。 目前有一些关于该功能的使用问题，      相关 [issue](https://github.com/GoogleChrome/puppeteer/issues/1350)
1. 如何上传文件？
    通过elementHandle.uploadFile(...filePaths) 接口。 目前只支持 input type="file" 类
    型的文件提交。 相关[issue](https://github.com/GoogleChrome/puppeteer/issues/1332)
1. 如何获得页面html代码？
      通过 [page.content()](https://github.com/GoogleChrome/puppeteer/blob/v0.13.0/docs/api.md#pagecontent)
1. 如何关闭javascript弹框
    通过 dialog.accept, 相关 [issue](https://github.com/GoogleChrome/puppeteer/issues/13)
    ```js
        page.on('dialog', dialog => {
            dialog.accept('test');
        });
    ```
1. 如何监控页面的网络请求？
    ```js
    const page = await browser.newPage();
    await page.setRequestInterceptionEnabled(true);

    page.on('request', request => {
      request.continue(); // pass it through.
    });

    page.on('response', response => {
      const req = response.request();
      console.log(req.method, response.status, req.url);
    });
    ```
1. 如何输入内容？
    方法1 page.type
    ```js
    // ...
    await page.focus('#lst-ib')
    page.type('China')
    // ...
    ```
    方法2 page.evaluate 后 element.value =
    ```js
    await page.evaluate((a, b) => {
       document.querySelector('#a').value = a;
       document.querySelector('#b').value = b;
       document.querySelector('#c').click();
     }, a, b);
    ```
1. 如何在页面中不同的Frame中切换
    通过page.frames()获得frame的数组，使用 iframe.$ 来获得对应frame中的handle
   例如：
   ```js
   const browser = await puppeteer.launch({headless: false});
   const page = await browser.newPage();
   await page.setContent('<iframe></iframe>');

   // the page.frames()[0] is always a main frame.
   const iframe = page.frames()[1];
   // fetch the body element of the iframe
   const body = await iframe.$('body');
   // ...
   // do something with `body`..
   // ...
   browser.close();
   ```
  1. 获取element中的自定义属性值
    通过page.evaluate 然后使用object.getAttribute

      await page.evaluate( (obj) => {
        return obj.getAttribute('data-src');
      }, imgurlEle);
