---
title: 在nuxtjs中组织api调用
categories:
  - front-end
img: ../../coverImages/nuxt-axios.png
tags:
  - nuxt
typora-root-url: ..\..
date: 2020-04-20 10:58:44
---





对与api的组织调用，nuxtjs的官方文档都是`axios.get('https://jsonplaceholder.typicode.com/users')`这样的格式。简单的用一用，也问题不大，但是项目一大之后，对请求url的管理，前置，后置拦截器的处理，就比较麻烦了。



### 拦截器

拦截器的设置相对简单，官方文档也有介绍。

这里传入的参数是`context`,`$axios, store`都是挂载在它上面的属性。

不要使用 `import axios from 'axios'`的形式,在`nuxt`中，`axios`可以在服务端，和客户端两种环境执行，框架已经对这两种情况做了封装，直接在插件目录引用`axios`就可以

```javascript
// plugins/axios.js

export default function({ $axios, store }) {
  $axios.onRequest((config) => {
    console.log('request')
    store.commit('showloading', true)
    return config
  })

  $axios.onResponse((response) => {
    store.commit('showloading', false)
    if (response.status === 200) {
      return response
    }
  })

  $axios.onError((error) => {
    store.commit('showloading', false)
    return Promise.reject(error)
  })
}
```



### api 的组织和分离

但是怎么样分离`api`?



#### 1. 依赖注入的方式注入`axios`

通过函数参数的形式，把`axios`注入到上下文中，直接调用。

```javascript
// api/index.js
// 高阶函数
export default (axios) => () => ({
  index(params) {
    return axios.post('/word', params)
  }
})
```



####  2.使用`nuxtjs`插件注入`axios`

- 我们如何在整个Nuxt应用程序中**访问**`api`？
- 我们如何从Nuxt模块正确**传入** axios实例？

```javascript
// plugins/axios-api-plugin.js
import createApi from '~/api/index'
// 这里ctx也可以访问到store
export default (ctx, inject) => {
  // 注入上下文
  // 挂载到vue实例上面 (组件中使用：this.$api)
  const apiAxios = createApi(ctx.$axios)
  inject('api', apiAxios())
}
```

再把上面的代码作为插件配置到`plugins`中就可以像这样调用api：

```javascript
// 客户端 
await this.$api.index(params)
// 服务端 asyncData /fetch
await ctx.app.api.index(params)
```

这样封装之后，模块清晰，可维护性大大增加。



### 可运行示例：

https://github.com/justOneWord/one_word_web



#### 参考资料：

https://blog.lichter.io/posts/nuxt-api-call-organization-and-decoupling/

https://codesandbox.io/s/github/manniL/nuxt-decouple-and-organize-api-calls

https://zh.nuxtjs.org/api/context



