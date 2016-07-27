---
layout: post
title: phantomjs学习笔记
date: 2016-07-25 18:00:00 +0800
categories: javascript
tags: javascript
---

因为项目的需要，需要使用无壳浏览器来抓取一些js生成的网页内容。因此学习到了[phantomjs](http://phantomjs.org/)的用法。

所谓无壳浏览器（Headless browser），指的是没有图形用户界面的浏览器。它具有浏览器的页面解析和js代码执行的功能，并提供了一些API用于网页的自动化控制。这在自动化测试和获取页面Ajax异步获取的内容时非常有用。

下面是一些来自[官方文档](http://phantomjs.org/documentation/)的例子。

## Hello world!
创建一个包含两行代码的文本文件：

```javascript
console.log('Hello, world!');
phantom.exit();
```

保存为`hello.js`,然后在命令行而不是REPL中运行它。

REPL是一个简单交互的编程环境。具体可以在[这里](http://phantomjs.org/repl.html)阅读文档。

在命令行运行该脚本`phantomjs hello.js`，会输出`Hello, world!`.

注意在脚本中调用`phantom.exit()`, 否则脚本程序不会终止运行。

## 页面加载
一个页面可以通过创建一个web page对象来进行加载分析和显示。

下面的脚本演示了最简单的web page对象的用法。它加载example.com并将页面保存为截图`example.png`, 该截图保存在脚本所在的目录。

```javascript
var page = require('webpage').create();
page.open('http://example.com', function(status) {
  console.log("Status: " + status);
  if(status === "success") {
    page.render('example.png');
  }
  phantom.exit();
});
```

由于phantomjs的渲染特性，phantomjs可以用来[对网页内容进行截图](http://phantomjs.org/screen-capture.html)。

下面的`loadspeed.js`用于测量网页的加载时间。

```javascript
var page = require('webpage').create(),
  system = require('system'),
  t, address;

if (system.args.length === 1) {
  console.log('Usage: loadspeed.js <some URL>');
  phantom.exit();
}

t = Date.now();
address = system.args[1];
page.open(address, function(status) {
  if (status !== 'success') {
    console.log('FAIL to load the address');
  } else {
    t = Date.now() - t;
    console.log('Loading ' + system.args[1]);
    console.log('Loading time ' + t + ' msec');
  }
  phantom.exit();
});
```

使用下边的命令运行脚本：
`phantomjs loadspeed.js http://www.google.com`
它会输出类似下边的内容。
`Loading http://www.google.com Loading time 719 msec`

## 代码求值
为了执行网页上下文中的js代码，我们需要使用`evaluate`函数。执行代码是运行在一个沙盒中的，沙盒中的代码无法访问网页上下文以外的javascript对象和变量。`evaluate`可以返回一个对象，该对象必须是一个简单对象，不能包含函数和闭包。

下面是一个显示网页标题的例子。

```javascript
var page = require('webpage').create();
page.open(url, function(status) {
  var title = page.evaluate(function() {
    return document.title;
  });
  console.log('Page title is ' + title);
  phantom.exit();
});
```

`evaluate`内部代码网页上下文产生的任何控制台信息，默认都不会被打印出来。我们可以使用onConsoleMessage回调函数来覆盖默认行为。上面的例子可以重写为：

```javascript
var page = require('webpage').create();
page.onConsoleMessage = function(msg) {
  console.log('Page title is ' + msg);
};
page.open(url, function(status) {
  page.evaluate(function() {
    console.log(document.title);
  });
  phantom.exit();
});
```

由于网页内部的js代码执行过程和在真正的网页浏览器中并没有什么区别，标准的DOM方法和CSS选择器都可以在`evaluate`内部很好的使用。这使得phantonjs适用于执行各种网页自动化测试。

## 网络请求和响应
当一个网页从远程服务器请求一个资源，请求和响应可以通过`onResourceRequested` 和 `onResourceReceived`回调函数进行追踪。这在示例[netlog.js](https://github.com/ariya/phantomjs/blob/master/examples/netlog.js)中有演示。

```javascript
var page = require('webpage').create();
page.onResourceRequested = function(request) {
  console.log('Request ' + JSON.stringify(request, undefined, 4));
};
page.onResourceReceived = function(response) {
  console.log('Receive ' + JSON.stringify(response, undefined, 4));
};
page.open(url);
```
