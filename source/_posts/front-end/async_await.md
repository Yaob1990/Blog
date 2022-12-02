---
title: async await 的终极封装
categories:
  - front-end
img: ../../coverImages/javascript_async_await.jpeg
tags:
  - promise
typora-root-url: ..\..
date: 2020-08-06 10:00:00
---

## async await 的终极封装

在看同事代码时候，看到下面这一段，有点意思。

```js
let [res,error] = await getList()
if(error){
  // 错误处理
}
// 业务处理
```

注意，这里代码没有使用try catch，而是把错误处理封装成了数组形式。

继续，看看他的代码是怎么写的：

```js
export function promieWapper(promise) {
  return new Promise((resolve, reject) => {
    promise.then(res => {
      resolve([res, undefined]);
    }).catch(err => {
      resolve([undefined, err]);
    });
  })
}

getList = async () => promieWapper(getData());
```

看起来也不是很复杂，就是在promise 外面在套一层promise，返回结果，放到一个数组中。

很巧妙的实现了，错误的处理，使用时候，直接进行数组的判断，不需要去写丑陋的 try catch

## 优化

仔细看一下上面的代码，其实有点小问题，内部返回的就是promise，还有必要再嵌套一层 promise ？

```js
export function promieWapper(promise) {
  return promise.then(res => {
      return [res, undefined];
    }).catch(err => {
      retirn [undefined, err];
    });
}
```

这样写感觉更清爽一点， then 返回的部分也还是 promise，异常也会被 catch 捕获到。

## 添加函数类型（ts）

```typescript
async function promieWapper<T, U = Error>(promise: Promise<T>): Promise<[U, undefined] | [null, T]> {
  return promise.then<[null, T]>((data) => ([null, data]))
    .catch<[U, undefined]>((error: U) => [error, undefined])
}
```

node 中一般 Error 会放在第一个参数位置，这里参考 node 优先处理错误。

## 3年前...

我觉得我同事牛逼极了，能想出这么天才的方法！

直到我看到了这个库：[await-to-js](https://github.com/scopsy/await-to-js)

上面的最终写法，也是参考了这个库。这个库 2.3m/month 的下载量，还是3年前写的，写好基本没有更新过，真牛逼！



