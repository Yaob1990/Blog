---
title: js 中 setTimeout 计时器最大值
date: 2021-03-23 07:00:00
categories: 前端
img: ../../coverImages/time.jpeg
tags:
  - js
---

项目中踩坑，记录下。
没有意识到 `setTimeout/setInterval` 计时器有最大值问题。

```js
setTimeout(() => {
  console.log(1111)
}, 2 ** 31)
```
这部分代码，不会等到计时器结束，而是会被会被立即执行。

## 原因
setTimeout/setInterval 使用 `int32` 存储延时参数值，也就是说最大延时值是 `2^31-1`(约为24.85天) ，超过这个值会被立即执行。

## 解决方案
### web
在 web 页面中很少有需要这么大延时值得情况，根据业务场景，超过1小时，不做定时器设置，1小时以内，做定时器设置。

### node
服务端，确实有类似的情况，比如1月一次的定时任务。建议使用 `corn` 代替 `setTimeout`，也会比 `setTimeout` 更加准确。

