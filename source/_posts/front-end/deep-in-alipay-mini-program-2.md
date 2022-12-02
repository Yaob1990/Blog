---
title: 深入理解支付宝小程序 -- sjs 篇
categories:
  - front-end
img: ../../coverImages/mini-alipay.jpeg
tags:
  - 小程序
date: 2021-07-31 22:00:00
---

在平时的开发中，我们很少会主动去使用 `sjs`，程序能够运行，不加班，已经是极好的事情了:)。但我们这个系列不可以，深入原理部分，必须对每一个细节都了若指掌，差之毫厘谬以千里。

## sjs 定义
以下是官方定义：
> SJS（safe/subset javascript）是小程序一套自定义脚本语言，可以在 AXML 中使用其构建页面结构。 SJS 是 JavaScript 语言的子集，与 JavaScript 是不同的语言，其语法并不与 JavaScript 一致，请勿将其等同于 JavaScript。

- sjs 中只支持使用 import、export 管理模块依赖。
- sjs 只能定义在 .sjs 文件中。然后在 axml 中使用 <import-sjs> 标签引入。
- sjs 可以调用其他 sjs 文件中定义的函数。
- sjs 是 JavaScript 语言的子集，请勿将其等同于 JavaScript。
- sjs 的运行环境和其他 JavaScript 代码是隔离的， sjs 中不能调用其他 JavaScript 文件中定义的函数，也不能调用小程序提供的 API。
- sjs 函数不能作为组件事件回调。
- sjs 不依赖于基础库版本，可以在所有版本小程序中运行。
- sjs 可以响应事件

简单理解：`sjs` 是js的子集，运行在受限的容器中，一般不能修改业务数据，可以响应事件。

## 深入分析 sjs

### 不同的报错
<!--我们在第0篇分析过`mini-pkg-builder`文件，这里我们换一个思路。-->
我们尝试在业务代码和 `sjs` 中使用 `eval` 函数，两者提示不一样：

```js
// af-appx.worker.min.js
TypeError: eval is not a function

// VM194 index.html:3 Module build failed (from /snapshot/code-repo/out/target/bundle/node_modules/@ali/antcube-thread-loader/lib/cjs.js):
SyntaxError: identifier(eval) is disallowed in sjs
```

注意这里两个报错是不一样的！在`af-appx.worker.min.js` 中，eval 并全局指向了 `undefined`, 在 sjs 中，`eval` 是不被运行使用。不被允许，并不是不存在的意思，也就是这个方法是存在的，只是我不让你使用。

### 双线程架构
小程序的架构中 js 一般在独立线程中执行，页面的渲染在 `webview` 中执行，两者通过 `jsbridge` 进行通信。那么 `sjs` 在哪里执行？
这一次我们使用miniu构建我们的小程序，我们分析小程序打包的代码：
目录结构
.
├── appConfig.json
├── assets
│   └── logo.png
├── index.html
├── index.js
├── index.worker.html
├── index.worker.js
└── manifest.json

index.html 中文件的加载顺序
```html
<body>
<script src="index.js"></script>
<script>
    window.bootstrapApp({
      worker: 'index.worker.js?version=1627720007775a8392433130335686',
      onReady: onReady,
    });
</script>
```
页面在加载时候，会先加载 `index.js` 文件，然后启动 works，执行用户的业务代码。
我们的 `sjs` 代码被打入 `index.js` 中，这里我把 `a:222`,改成了`a=eval(333)`,页面正常显示。
![](/images/16277221107831.jpg)

`index.sjs` 应该并不是用户逻辑的代码，而是服务于`webview`部分的代码，所有和webview 相关的代码都被放到了这里，它的运行环境就是 `webview`。
我们继续尝试，`a` 改成 `window.innerHeight`,页面运行成功，显示 `595` 高度，验证了我们的猜想，这里可以访问到完整的 `webview` 环境。

## 安全
小程序各种安全体系下，`sjs` 比较容易产生风险。
小程序打包需要执行 `mini-pkg-builder` 程序，我们在本地预览，远程调试时候，都是他在做小程序的打包工作，这是一个加密的 unix 可执行文件，尝试反编译，无果。

我们小程序上传时候，会在服务端的容器中执行 `mini-pkg-builder` 程序。保证了程序本身不会被用户影响，即使本地尝试修改这个文件，也不会影响到正式发布。

然而，攻与防从来都是相对的，这里会不会存在逻辑bug，就不得而知了，有趣的你是不是跃跃欲试了。

## 参考

- [小程序底层实现原理及一些思考 - 知乎](https://zhuanlan.zhihu.com/p/81775922)
- [小程序底层实现原理及一些思考（2） - 知乎](https://zhuanlan.zhihu.com/p/121815358)
- [独家！支付宝首次披露其小程序技术架构](https://mp.weixin.qq.com/s/PX7b_qV6tYKnN3ecoz9Ehw)
- [利用sourceMap定位错误实践](https://juejin.cn/post/6882265367251517447)
- [微信小程序开发专区 | 薛定喵君](http://tiaocaoer.com/xcx_study/)