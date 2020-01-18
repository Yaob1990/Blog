---
title: 从Axios源码学习Promise的妙用
date: 2020-01-5 20:15:00
categories: 前端
img: ../../coverImages/axios.jpg
tags:
  - axios
  - promise
typora-root-url: ..\..
typora-copy-images-to: ..\..\images
---

惭愧，虽然一直在使用`Promise/async/await`，但是自己其实对`Promise`并不是特别理解，导致这次遇到问题时候，一直不是特别明白。

---

## 需求分析

> 需求：前端定时刷新 token 接口

分析：看起来不复杂，但是如果考虑到 token 有时间限制，超时不可用，同时发起多个请求，怎么样在更新 token 的时候，延迟其他请求的发送，这就有点复杂了。

接口我是用 axios 去请求，这部分功能应该都会在拦截其中实现。我遇到的难点，也就是如何 hold 住请求，等待 token 刷新，释放请求。

```js
import axios from "axios";

// 从localStorage中获取token，token存的是object信息，有tokenExpireTime和token两个字段
function getToken() {
  let tokenObj = {};
  try {
    tokenObj = storage.get("token");
    tokenObj = tokenObj ? JSON.parse(tokenObj) : {};
  } catch {
    console.error("get token from localStorage error");
  }
  return tokenObj;
}

function refreshToken() {
  // instance是当前request.js中已创建的axios实例
  return instance.post("/refreshtoken").then(res => res.data);
}

// 给实例添加一个setToken方法，用于登录后方便将最新token动态添加到header，同时将token保存在localStorage中
instance.setToken = obj => {
  instance.defaults.headers["X-Token"] = obj.token;
  window.localStorage.setItem("token", JSON.stringify(obj)); // 注意这里需要变成字符串后才能放到localStorage中
};

instance.interceptors.request.use(
  config => {
    const tokenObj = getToken();
    // 添加请求头
    config.headers["X-Token"] = tokenObj.token;
    // 登录接口和刷新token接口绕过
    if (
      config.url.indexOf("/rereshToken") >= 0 ||
      config.url.indexOf("/login") >= 0
    ) {
      return config;
    }
    if (tokenObj.token && tokenObj.tokenExpireTime) {
      const now = Date.now();
      if (now >= tokenObj.tokenExpireTime) {
        // 立即刷新token
        if (!isRefreshing) {
          console.log("刷新token ing");
          isRefreshing = true;
          refreshToken()
            .then(res => {
              const { token, tokenExprieIn } = res.data;
              const tokenExpireTime = now + tokenExprieIn * 1000;
              instance.setToken({ token, tokenExpireTime });
              isRefreshing = false;
              return token;
            })
            .then(token => {
              console.log("刷新token成功，执行队列");
              requests.forEach(cb => cb(token));
              // 执行完成后，清空队列
              requests = [];
            })
            .catch(res => {
              console.error("refresh token error: ", res);
            });
        }
        const retryOriginalRequest = new Promise(resolve => {
          requests.push(token => {
            // 因为config中的token是旧的，所以刷新token后要将新token传进来
            config.headers["X-Token"] = token;
            resolve(config);
          });
        });
        return retryOriginalRequest;
      }
    }
    return config;
  },
  error => {
    // Do something with request error
    return Promise.reject(error);
  }
);

// 请求返回后拦截
instance.interceptors.response.use(
  response => {
    const { code } = response.data;
    if (code === 1234) {
      // token过期了，直接跳转到登录页
      window.location.href = "/";
    }
    return response;
  },
  error => {
    console.log("catch", error);
    return Promise.reject(error);
  }
);

export default instance;
```

核心代码：

```js
// 1. 保存 pendding 状态的 Promise 到数组 requests 里面
const retryOriginalRequest = new Promise(resolve => {
  requests.push(token => {
    config.headers["X-Token"] = token;
    resolve(config);
  });
});
// 2. 执行 pendding 的promise ，状态转为 resolve
requests.forEach(cb => cb(token));
```

参考文章：[axios 如何利用 promise 无痛刷新 token（二）](https://juejin.im/post/5dcac7686fb9a04a9e37b595)

我在如何保存`Promise`,后续如何释放`Promise` 这里卡住了，他的实现比较巧妙。

1. 保存请求的 config 的 Promise 到数组中
2. 在刷新 token 之后，再执行修改 Promise 状态

然鹅，你以为我懂了...其实我还是懵逼的...

![奴家不会啊](/images/1-1578487578225.png)

## 如何理解 Promise

> ```js
> const retryOriginalRequest = new Promise(resolve => {
>   requests.push(token => {
>     // 因为config中的token是旧的，所以刷新token后要将新token传进来
>     config.headers["X-Token"] = token;
>     resolve(config);
>   });
> });
> return retryOriginalRequest;
> ```
>
> 这里返回的是 promise？？？
>
> 一般我们不是返回的 config 吗？返回 Promise 作甚？？？

在学习`axios`源码的过程中，找到了答案。

```js
export default class InterceptorManager<T> {
  private interceptors: Array<Interceptor<T> | null>

  constructor() {
    this.interceptors = []
  }
// 使用拦截器
  use(resolved: ResolvedFn<T>, rejected?: RejectedFn): number {
    this.interceptors.push({
      resolved,
      rejected
    })
    return this.interceptors.length - 1
  }

  forEach(fn: (interceptor: Interceptor<T>) => void): void {
    this.interceptors.forEach(interceptor => {
      if (interceptor !== null) {
        fn(interceptor)
      }
    })
  }

  eject(id: number): void {
    if (this.interceptors[id]) {
      this.interceptors[id] = null
    }
  }
}
```

执行流程：

```js
 //  链式调用
    const chain: PromiseChain<any>[] = [
      {
        resolved: dispatchRequest,
        rejected: undefined
      }
    ]
    this.interceptors.request.forEach(interceptor => {
      //  后添加先执行
      chain.unshift(interceptor)
    })
    this.interceptors.response.forEach(interceptor => {
      //  先添加先执行
      chain.push(interceptor)
    })

    let promise = Promise.resolve(config)
    while (chain.length) {
      const { resolved, rejected } = chain.shift()!
      promise = promise.then(resolved, rejected)
    }

    return promise
```

基本流程：

1. 使用 use 方法，使用拦截器，拦截器被 push 到`interceptors`中进行管理
2. 合并基本配置之后，设置调用链`chain`,请求拦截器，从后往前`unishift`到 chain 中，响应拦截器被`push`到`chain`中，中间则是，调用执行方法，也就是`dispatchRequest`
3. 这样就保证了调用顺序，请求拦截器 ->执行调用方法 ->响应拦截器
4. 数据的流转，全部是通过`promise`传递，这里`while`方法，还是递归

## 理解 axios 中的 Promise

再回头看官方的示例：

```js
// Add a request interceptor
axios.interceptors.request.use(
  function(config) {
    // Do something before request is sent
    return config;
  },
  function(error) {
    // Do something with request error
    return Promise.reject(error);
  }
);
```

error 时候，返回 reject，正常时候，只返回了 config。在下一步的执行中，实际变成了

```js
promise = promise.then(config, rejected);
```

虽然没有使用`resolve(config)`的形式，但是实际上是一样的，`promise.then()`中`return value` 会被包装成`Promise.then(return value)`

再看我的困惑，就不是问题了，`return Promise` 出去，会被后续的`Promise` 捕获，并被 hold，直到`resolve`执行之后，请求继续执行~

通过一个问题去阅读源码，再去理解基础知识。有的知识点是知道了，有的是会用了，然而，并没有真正的理解，想举一反三，难呐 😭
