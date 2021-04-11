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
```js
constructor (options = {}) {
    // webpack.DefinePlugin 配置
    if (__DEV__) {
      assert(typeof Promise !== 'undefined', `vuex requires a Promise polyfill in this browser.`)
      assert(this instanceof Store, `store must be called with the new operator.`)
    }

    const {
      plugins = [],
      //  严格模式，不允许在生产环境下开启，会导致性能损失
      strict = false
    } = options

    // store internal state
    this._committing = false
    this._actions = Object.create(null)
    this._actionSubscribers = []
    this._mutations = Object.create(null)
    this._wrappedGetters = Object.create(null)
    this._modules = new ModuleCollection(options)
    this._modulesNamespaceMap = Object.create(null)
    this._subscribers = []
    this._makeLocalGettersCache = Object.create(null)

    // bind commit and dispatch to self
    const store = this
    const { dispatch, commit } = this
    // 绑定 dispatch 指向自己
    this.dispatch = function boundDispatch (type, payload) {
      return dispatch.call(store, type, payload)
    }
    this.commit = function boundCommit (type, payload, options) {
      return commit.call(store, type, payload, options)
    }

    // strict mode
    this.strict = strict

    // 定义 根state
    const state = this._modules.root.state

    // init root module.
    // this also recursively registers all sub-modules
    // and collects all module getters inside this._wrappedGetters
    // 模块入口，内部注册模块，注册子模块
    installModule(this, state, [], this._modules.root)

    // initialize the store state, which is responsible for the reactivity
    // (also registers _wrappedGetters as computed properties)
    // 初始化 state
    resetStoreState(this, state)

    // apply plugins
    // 一般都是通过 store.subscribe 进行订阅，在 mutation 之后执行
    plugins.forEach(plugin => plugin(this))

    const useDevtools = options.devtools !== undefined ? options.devtools : /* Vue.config.devtools */ true
    if (useDevtools) {
      devtoolPlugin(this)
    }
  }
```

在构造函数中，定义了一系列的参数，参数内容基本都是纯粹空 object（Object.create(null)），可以理解成在构造函数中，对所有的 commit，dispatch等进行了一次收集，触发时候，直接从这边取。
dispatch commit，是外界直接调用的函数，一般使用方式`this.$store.dispatch('increment',5);`,为了确保指向store本身，这里用 `call` 做了一次 this 绑定，否则这里就会指向了 `this` 也就是 vue 本身了。

## 模块化
## commit 
```js
commit (_type, _payload, _options) {
    // check object-style commit
    const {
      type,
      payload,
      options
    } = unifyObjectStyle(_type, _payload, _options)

    const mutation = { type, payload }
    // 取出对应的 mutations
    const entry = this._mutations[type]
    if (!entry) {
      // 没有entry，开发模式报错
      if (__DEV__) {
        console.error(`[vuex] unknown mutation type: ${type}`)
      }
      return
    }
    this._withCommit(() => {
      // 执行entry，这里为啥用 foreach，会存在 entry 数组的情况？
      // 测试之后，这里应该只有1个数组内容的情况，不会出现多个数组元素
      entry.forEach(function commitIterator (handler) {
        handler(payload)
      })
    })
  }
  ```
  
  commit 部分看起来比较简单，通过函数参数获取 type、payload、options、，取出mutation，执行mutation。
  这里有个疑问，为什么entry是forEach执行的，这里一般情况下，应该entry内部只有一个元素,[mutation]，这样的形式。
  ```js
  entry.forEach(function commitIterator (handler) {
        handler(payload)
  })
  ```
  

## dispatch
```js
dispatch(_type, _payload) {
  // check object-style dispatch
  const {
    type,
    payload
  } = unifyObjectStyle(_type, _payload)

  const action = { type, payload }
  const entry = this._actions[type]

  try {
    // before
    this._actionSubscribers
      .slice() // shallow copy to prevent iterator invalidation if subscriber synchronously calls unsubscribe
      .filter(sub => sub.before)
      .forEach(sub => sub.before(action, this.state))
  } catch (e) {
  }

  const result = entry.length > 1
    ? Promise.all(entry.map(handler => handler(payload)))
    : entry[0](payload)
  // dispatch 最终返回的是一个 promise
  return new Promise((resolve, reject) => {
    result.then(res => {
      try {
        // after
        this._actionSubscribers
          .filter(sub => sub.after)
          .forEach(sub => sub.after(action, this.state))
      } catch (e) {
      }
      resolve(res)
    }, error => {
      try {
        this._actionSubscribers
          .filter(sub => sub.error)
          .forEach(sub => sub.error(action, this.state, error))
      } catch (e) {
      }
      reject(error)
    })
  })
}
```
dispatch 相对 commit 就要复杂的多了。


## 插件体系
