---
title: 重构 uniapp 项目(1)：uniapp 中的数据流机制
categories:
  - front-end
img: ../../coverImages/refactoring.jpeg
tags:
  - 小程序
date: 2021-07-20 23:30:00
---

  uniapp 遵循 vue 的语法，可以使用 vuex 等生态。
  
## 数据操作猜想
vue 在 H5 中，不管引入多少概念，虚拟 dom，diff 算法，它最新依然是需要操作dom，`document.getElementById("p1").innerHTML = "hello kitty!";` 类似这样的修改一定会在 vue的源码中出现。
小程序 js core 和 webview 的通过 `JSBridge` 进行通信，也就是 setData 这个函数。`JSBridge` 传递的是字符串，页面层，进行一次 diff 之后，渲染出数据。

## uniapp setData 的调用

  
  
## 参考
  - [小程序插件 - uni-app官网](https://uniapp.dcloud.io/component/mp-weixin-plugin)
  - [fix: 支付宝小程序 life-follow 事件去掉修饰符 by Yaob1990 · Pull Request #2759 · dcloudio/uni-app](https://github.com/dcloudio/uni-app/pull/2759)
  - [fix:(mp-alipay): 支付宝小程序平台增加独有内置组件判断 #2410#issuecomment-878974559 · dcloudio/uni-app@92d682a](https://github.com/dcloudio/uni-app/commit/92d682a11a27d8fda573555512fff69a94a94e40)
  - [uni-app是如何构建小程序的？](https://juejin.cn/post/6968438754180595742#heading-22)