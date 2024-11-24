---
title: 深入理解支付宝小程序 -- web Components、rpx 篇
categories:
  - front-end
img: ../../coverImages/mini-alipay.jpeg
tags:
  - 小程序
date: 2021-07-31 21:30:00
---

## web Components
### 微信小程序
调试ide版本：v1.02.1907300
微信小程序 ide 使用 nw.js 开发，我们可以尝试打开它的控制台
```js
document.getElementsByTagName('webview')[0].showDevTools(true,null)
```
![](/images/16272568175378.jpg)

我们拿到了微信小程序的渲染层页面，注意body中的标签 `wx-view`、`wx-open-data`，这看起来是一种类似 `web Components`的实现，这是一种浏览器原生支持的组件形式。

### 支付宝小程序
调试 ide 版本：2.1.9
支付宝小程序是使用 Electron 开发的，我们也可以打开它的控制台：
```js
document.getElementsByTagName('webview')[1].openDevTools()
```
外部套着模拟器，内部是`iframe`，这里应该就是小程序本身代码，也就是 `iframe` 部分
![](/images/16272573416726.jpg)

这里并没有和小程序类采用了组件的形式，而是`原生标签 + class`的形式。尝试全局修改`a-view`的样式，竟然真的生效了。
进一步测试发现，标签选择器会被编译成`.a-*`的形式，不局限于已有的标签，如果是`a`标签也会这样编译。

### 分析
微信小程序的标签需要基础库去解析，支付宝这里似乎是没有的，他直接编译成了`html`,并且`html`没有做到样式隔离，能够被外部影响。如果无意中写了类似标签，还是非常难以排查的。

## rpx
rpx 是小程序平台的响应式单位。

> rpx（responsive pixel）可以根据屏幕宽度进行自适应，规定屏幕宽为 750rpx。以 Apple iPhone6 为例，屏幕宽度为 375px，共有 750 个物理像素，则 750 rpx = 375 px = 750 物理像素，1rpx = 0.5 px = 1 物理像素。


### 微信小程序
viewport 设置
```js
<meta name="viewport" content="width=device-width,user-scalable=no,initial-scale=1,maximum-scale=1,minimum-scale=1">
```
rpx 转换：

```js
var BASE_DEVICE_WIDTH = 750;
var isIOS = navigator.userAgent.match('iPhone');
var deviceWidth = window.screen.width || 375;
var deviceDPR = window.devicePixelRatio || 2;
var checkDeviceWidth =
  window.__checkDeviceWidth__ ||
  function () {
    var newDeviceWidth = window.screen.width || 375;
    var newDeviceDPR = window.devicePixelRatio || 2;
    var newDeviceHeight = window.screen.height || 375;
    if (window.screen.orientation && /^landscape/.test(window.screen.orientation.type || '')) newDeviceWidth = newDeviceHeight;
    if (newDeviceWidth !== deviceWidth || newDeviceDPR !== deviceDPR) {
      deviceWidth = newDeviceWidth;
      deviceDPR = newDeviceDPR;
    }
  };
checkDeviceWidth();
var eps = 1e-4;
var transformRPX =
  window.__transformRpx__ ||
  function (number, newDeviceWidth) {
    if (number === 0) return 0;
    number = (number / BASE_DEVICE_WIDTH) * (newDeviceWidth || deviceWidth);
    number = Math.floor(number + eps);
    if (number === 0) {
      if (deviceDPR === 1 || !isIOS) {
        return 1;
      } else {
        return 0.5;
      }
    }
    return number;
  };
```

### 支付宝小程序
viewport 设置
```js
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=0, viewport-fit=cover">
```
根字体设置:
```js
// af-appx.min.js
function Sy() {
        _y = document.documentElement.clientWidth, xy = document.documentElement.clientHeight, wy = Math.min(_y, xy);
        var t = 100 * _y / 750 + "px";
        document.documentElement.style.fontSize !== t && (document.documentElement.style.fontSize = t)
    }
```
编译之后的样式文件：
```javascript
/*ACSSCompileContext:{"atImports":[]}*/var internal_style;

internal_style = ".title {background:red;font-size:0.5rem}.ttt {font-size:0.5rem}.hair {height:1px}.a-view {background:orange}.ttt .a-a {font-size:100px!important}#id {font-size:1rem}";
export default internal_style;
```


### 分析
微信小程序，rpx 被转换成了px。支付宝小程设置了根字体，rpx 最终被转换成了 rem。支付宝根字体的转换找到了代码，rpx 转 px 没有找到代码，猜测，应该是在编译过程中已经进行了转换，我分析的运行时部分没有这部分代码。

## 总结
貌似一样的众多小程序，仔细分析，发现还是有很多很多的不同，某些不同，还是基础架构层面的不同，深入分析，应该可以挖掘出更多有意思的东西。

## 参考

- [小程序底层实现原理及一些思考 - 知乎](https://zhuanlan.zhihu.com/p/81775922)
- [小程序底层实现原理及一些思考（2） - 知乎](https://zhuanlan.zhihu.com/p/121815358)
- [独家！支付宝首次披露其小程序技术架构](https://mp.weixin.qq.com/s/PX7b_qV6tYKnN3ecoz9Ehw)
- [利用sourceMap定位错误实践](https://juejin.cn/post/6882265367251517447)
- [微信小程序开发专区 | 薛定喵君](http://tiaocaoer.com/xcx_study/)