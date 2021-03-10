---
title: vuex 4.0 源码解析
date: 2021-03-10 22:40:00
categories: 前端
img: ../../coverImages/vuex.png
tags:
  - 源码
---

vuex 源码短小精悍，很短时间就可以大致浏览一遍，值得学习。在后续看其他框架的过程中，也发现很多框架都有参考 vuex（比如 herbjs 的插件体系）。

## 猜想
1. 一定有一个地方集中存放所有的数据（state），可以挂载到 window 上面（污染全局），或者挂载到 vue 的原型上面。
2. mutation action getter 这些方法，其实都是获取或者修改 state 的值，这些方法和 state 刚好可以组成一个 类（class），类封装对 state 状态的操作。

## 如何安装
4.0 版本适配了 composition api，提供了导出方法 createStore，通过它可以创建 store。
源码：
```js
export function createStore (options) {
  return new Store(options)
}
```
可以看到，这里只是简单的对 store 进行了一次 new 操作，返回了 store 这个类。
进一步查看，我们看到 install 方法，通过暴露install方法，按照 vue3 的插件规则进行安装，也就是挂载到了 vue3 的原型上面。
```js
export class Store {
    // 暴露给 vue3 进行安装
  install (app, injectKey) {
    app.provide(injectKey || storeKey, this)
    app.config.globalProperties.$store = this
  }
}
```

## store 初始化


## 模块化
## commit 

## dispatch

## 插件体系
