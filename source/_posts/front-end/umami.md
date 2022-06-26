---
title: umami 源码分析
date: 2022-06-26 22:53:02
categories: 前端
tags:
  - umami
  - 源码
img: ../../coverImages/umami.png
---

umami 不使用 cookie 、localstorage 实现了 uv 识别，符合最新的隐私规范，代码也比较精简。花了一点时间，研究了这部分的实现，还是很有意思的。

## 技术框架：
next.js + mysql/postgresql
整体看下来，nextjs 准备了很多的约定，比如说 api 目录在 `pages/api/`下面，比如说，`pages/api/user.js`，nextjs 框架有大量这样的约定。

个人还是不太习惯这样的框架，总有种不伦不类的感觉，一些小项目可以这样搞，大型项目，还是需要明确的代码分成，封装。

## 用户识别
uv 的核心是去做用户识别，一般会本地存储一个随机的id，每次页面路由变化，上报给后台。
而 umami 为了符合一系列的隐私规范，并没有这么做，没有使用 cookie或者 localStorage 。
上报路径"pages/api/collect.js"，最终会产生 session 并且通过 session 返回 token 。session 生成方法 `getSession`

```js
const session_uuid = uuid(website_id, hostname, ip, userAgent);
session = await createSession(website_id, {
    session_uuid,
    hostname,
    browser,
    os,
    screen,
    language,
    country,
    device,
});
```
这部分代码就比较清楚了，根据网站id，域名，ip，userAgent 生成 session_uuid,然后和数据库通信创建或者使用 session。
核心是根据一些列的变量生成一个不变的 uuid，后续用户再次进入页面，根据用户的这些参数，去数据库查询这个 uuid，就实现了用户识别。

里面也有一些其他的逻辑，比如跨域，忽略本地地址等，但是不影响我们对核心逻辑的理解。

## 上报脚本脚本
上报脚本位置：`tracker/index.js`
这个脚本很短，只有短短的225实现了上报功能。
```js
<script async defer data-website-id="914685a1-8993-4d8c-895b-929c8646e814" src="http://localhost:3000/umami.js"></script>
```
* async: async 脚本会在后台加载，并在加载就绪时运行。DOM 和其他脚本不会等待它们，它们也不会等待其它的东西。async 脚本就是一个会在加载完成时执行的完全独立的脚本。
* defer: 特性告诉浏览器不要等待脚本。相反，浏览器将继续处理 HTML，构建 DOM。脚本会“在后台”下载，然后等 DOM 构建完成后，脚本才会执行。
* data-website-id: 网站 id

## 挂载
```js
if (!window.umami) {
    const umami = eventValue => trackEvent(eventValue);
    umami.trackView = trackView;
    umami.trackEvent = trackEvent;

    window.umami = umami;
  }
```
方法都挂载到 window 上面，后续可以直接调用。

## 记录 pv uv
```js
if (autoTrack && !trackingDisabled()) {
    // 监听 pushState，replaceState 事件
    history.pushState = hook(history, 'pushState', handlePush);
    history.replaceState = hook(history, 'replaceState', handlePush);

    const update = () => {
      console.error('update');
      if (document.readyState === 'complete') {
        console.error('complete');
        trackView();

        if (cssEvents) {
          addEvents(document);
          observeDocument();
        }
      }
    };

    document.addEventListener('readystatechange', update, true);

    update();
  }
```
这里最终会在`document.readyState === 'complete'`时候，去做事件监听绑定等操作，发送第一次页面上报。
有一点不理解，为什么已经监听了`readystatechange`，还是又手动执行了一次`update()`
## 监听路由改变
```js
export const hook = (_this, method, callback) => {
  const orig = _this[method];
  return (...args) => {
    callback.apply(null, args);
    return orig.apply(_this, args);
  };
};
```

`history.pushState` hook劫持，为了在原生方法执行前，执行callback。这样实现了对原生 history 的监听。`handlePush` 方法会执行上报方法 trackView。

## 总结
整体代码比较简单，清晰，无侵入性的实现了网站统计。







