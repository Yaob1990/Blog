---
title: 深入理解支付宝小程序 -- event、jsBridge通信篇
categories:
  - front-end
img: ../../coverImages/mini-alipay.jpeg
tags:
  - 小程序
date: 2021-07-31 23:00:00
---

这篇我们将跟踪函数，尝试去分离出 `webview` 和 `worker` 的通信方式.

## 分析方法
![](/images/16277398312977.jpg)

在调试模式中，对按钮事件打断点，一层一层跟下去，不要纠结细节，只关注核心实现。


## 事件路径分析
![事件分析](/images/event.jpg)

事件的路径非常非常非常厂，图中省略了很多细节。简单来说，Dom 触发事件，被 `af-appx.min.js` 监听到，然后发送消息，这个消息被 `vendors.bundle.js` 接收到，一系列转换之后，发送给 `AlipayJSBridge`, 'af-appx.worker.min.js',`onMessage`接收到消息，进入业务代码流程，一系列转换之后，业务函数被调用，发送消息给`AlipayJSBridge`，页面重新渲染。


## 小程序模型分析
这里其实很明确了，我们说的 webview 层，不是只有 html，而是一套封装了`事件处理、dom 操作、页面渲染、diff 算法`的接口。通过 `AlipayJSBridge` 页面层获取数据，然后根据数据，做一整套的渲染流程，其中有大量的 `js`代码，同时`sjs` 也在这一层。
业务层用户的代码也并不是被直接执行，其中存在着一套公共的处理系统，由这套系统去做用户代码的调度，同时这层代码去做 `AlipayJSBridge` 的沟通事务。

## 如何保证事件的唯一性
事件的执行可能有快慢，先触发的并不一定先收到返回值，js 的异步特性，也不会允许线程阻塞去等待。小程序的处理很巧妙，一个 id 解决。
![](/images/16277406930646.jpg)

这是 `af-appx.min.js` 发送的数据，注意其中的 `i`, 它是一个递增的值，这样发送和收到的消息就可以找到彼此了。
这里的消息发送，最终走向了：

```js
AlipayJSBridge.call(t, e, n)
```

其中 t:'postmessage',
e:
![](/images/16277409067028.jpg)

 其中内部的 p，包含了传递的事件数据。
 
##  页面通信分析
从 webview 传递到逻辑层，都会触发 `this.port.postMessage`，我们尝试分离出触发 `this.port.postMessage` 的部分方法：

- `invokeHostTargetMethod`: 用户事件触发: `tap`
- `invokeHostPageMethod`: 页面级别事件：`reloadPage`、`getHostStartupParams`、`updateClientPerformance`、`reportProfile`
- `invokeHostPageEvent`: 页面触发事件：`onPageScroll`、`onReachBottom`、`onResize`
- `invokeHostBridgeCallProxy`: 页面路由相关：`navigateTo`、`switchTab`
- `invokeHMR`：热更新？
- `onPageNotFound`: 页面未发现

这些事件会触发相应的方法，调用 `AlipayJSBridge`, 实现页面和 `worker` 的通信。

## webview 渲染层
小程序的 webview 层可能部分使用了 React，在项目里面可以看到引用了 react 相关的库。不需要重新造轮子。react 本身也是一套非常精简的框架，只关注 ui 渲染。

网上找到相关资料：
> 我们选择 WebAssembly 作为虚拟 dom 的实现方向，WebAssembly 是一个新的 Web 标准，它定义了网页中的可执行代码的二进制格式和相应的类似汇编语言格式。他的目标是使执行代码几乎与本地机器代码一样快，它被用来作为 JavaScript 的补充，以加速 Web 应用程序的性能关键部分，所以我们使用 WebAssembly 技术重新实现了虚拟 dom 这块的核心代码，提升了小程序的页面渲染。

对 WebAssembly 不熟悉，暂时无法进一步深入。


## 参考

- [小程序底层实现原理及一些思考 - 知乎](https://zhuanlan.zhihu.com/p/81775922)
- [小程序底层实现原理及一些思考（2） - 知乎](https://zhuanlan.zhihu.com/p/121815358)
- [独家！支付宝首次披露其小程序技术架构](https://mp.weixin.qq.com/s/PX7b_qV6tYKnN3ecoz9Ehw)
- [利用sourceMap定位错误实践](https://juejin.cn/post/6882265367251517447)
- [微信小程序开发专区 | 薛定喵君](http://tiaocaoer.com/xcx_study/)