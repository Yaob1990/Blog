---
title: uniapp 中支付宝小程序自定义组件的调用
categories:
  - front-end
img: ../../coverImages/uniapp.jpg
tags:
  - 小程序
date: 2021-09-26 07:54:00
---

uniapp 这个缝合怪，挺厉害的，啥都能做，甚至你能够使用小程序的原生组件。
具体文档：
https://uniapp.dcloud.io/frame?id=%e5%b0%8f%e7%a8%8b%e5%ba%8f%e8%87%aa%e5%ae%9a%e4%b9%89%e7%bb%84%e4%bb%b6%e6%94%af%e6%8c%81

## 调用坑
写法1：
```js
<example @get="onGet" />
```
如果这样写，那么会走向 uniapp 的事件分发，但是支付宝这里事件的默认参数是 undefined，拿不到事件信息，所以这里会抛出 js 异常

写法2：
```js
<example onGet="onGet" />
```
使用小程序原生写法，会报 `event not found`,这是因为 uniapp 对事件有自己的封装，没有直接暴露给小程序，添加以下模板代码之后，可以调用成功：

```js
this.$scope.onGet = this.onGet.bind(this)
```

## 优化
如果使用组件比较多，模板代码可以放到 webpack loader 中去实现，思路就是模板中使用 `onGet` 写法的通过正则找出来，然后再 `onLoad()` 中挂载上去。

当我写好了 loader，并且 uniapp 编译成功之后，打开小程序编译器，迎接秋天的美好的时候

![](/images/16326137933990.jpg)

![a](/images/a.gif)

悲剧...一天就这样过去了...啊啊啊！！！