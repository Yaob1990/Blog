---
title: 抛弃 loadsh，封装更现代的防抖、节流
categories:
  - front-end
tags:
  - promise
date: 2021-11-16 22:22:22
---

## 一般意义上的防抖节流

### 函数防抖（debounce）

函数防抖，就是指触发事件后，在 n 秒后只能执行一次，如果在 n 秒内又触发了事件，则会重新计算函数的执行时间。简单的说，当一个动作连续触发，只执行最后一次。

### 函数节流（throttle）

限制一个函数在一定时间内只能执行一次

### 实现

lodash 里面提供了非常完备的实现，核心是使用定时器去延迟函数的执行

## 问题

1. 这样的防抖节流，只能屏蔽某一个时间段的特定操作。我们的点击，往往后面是一个网络请求，如果网络请求的事件长于这个时间，那么就可能出现防抖失效的情况。持续间隔点击按钮，可能会短时间内给后端发送多个请求，如果业务场无法保证幂等，就会出现问题。
2. ui 交互中，防抖实际上是延迟函数的执行，某些场景，会让人觉得卡顿，如果对交互的要求比较高，需要严格控制防抖的时间，一般不超过 200ms

## 如何防御

简单的实现，在业务代码中设置标志位，当有 promise 执行的时候，执行锁，等 promise 执行完毕，再释放锁。缺点，需要在业务中写很多重复的代码。

```javascript
const status = false

function btnClick() {
  if (status) {
    return
  } else {
    //   do something
    status = ture
  }
}
```

## 封装

我们可以参考防抖的实现，进行封装, 把函数用 Promise 进行包裹，然后设置标志位，就可以避免在业务中频繁设置标志位。
节流，我们可以设置函数执行的时候，同时执行一个定时器，也用 Promise 进行包装，当两个 Promise 都执行完毕的时候，再释放函数。

```typescript
/**
 * promise 节流函数版本
 * @param fn
 * @param time
 * @param delayTime
 * @constructor
 */
function PromiseThrottleFn(fn: Function, time = 0, delayTime = 0) {
  let status = ''
  return function () {
    if (status === 'lock') return
    status = 'lock'
    try {
      const data = Promise.all([
        // @ts-ignore
        fn.call(this, ...arguments),
        new Promise((resolve) => {
          setTimeout(resolve, time)
        }),
      ])
      // @ts-ignore
      return data[0]
    } finally {
      setTimeout(() => {
        status = ''
      }, delayTime)
    }
  }
}
```

函数我放到了个人的工具函数库中，可以直接通过 npm 安装：

```javascript
// 安装
npm install @aocoding/victorinox
// 使用
import { PromiseThrottleFn } from '@aocoding/victorinox'
```
