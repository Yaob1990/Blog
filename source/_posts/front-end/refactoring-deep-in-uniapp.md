---
title: 重构 uniapp 项目(1)：浅浅理解 uniapp
categories:
  - front-end
img: ../../coverImages/refactoring.jpeg
tags:
  - 小程序
date: 2021-07-20 23:30:00
---

  uniapp 在业内名气口碑都挺不错的，选择这个框架，能够让开发者快速出活，依托 vue 的生态，相关开发者也多，企业也容易招聘。然而小程序和web毕竟还是有点区别，某些时候还是需要对uniapp 有一定的理解，才能够顺利的搬砖。本篇，我们尝试解决 uniapp 的两个问题，一窥其内在机理。
  
## uniapp 小程序插件的使用
  
  uniapp 论坛有不少这样的帖子，支付宝插件在子组件无法使用。
  官方的用法是这样：
    1. 引入插件代码包: 使用插件之前开发者需要在manifest.json中的各平台对应的字段内声明使用的插件，具体配置参照所用插件的开发文档
  ```js
  // 支付宝小程序
    "mp-alipay": {
      "plugins": {
        "myPlugin": {
          "version": "*",
          "provider": "2019235609092837"
        }
      }
    }
  ```
  
  2. 在页面中使用
  
  ```js
  {
  "path": "pages/index/index",
  "style": {
    "mp-alipay": {
      "usingComponents": {
        "hello-component": "plugin://myPlugin/hello-component"
      }
    }
  }
}
  ```
  
看起来好像没啥坑，然鹅....编译之后你会发现，他在页面基本确实是引入了插件，但是组件内部并没有引入插件，于是你在组件内使用插件就报错了~
```js
<component>
    <plugin>
</component>
```
### 解决办法
解决方式倒也不复杂，既然在组件内部不给我使用插件，那么我就在组件内挖个 slot，然后在页面级别，给组件传递插件的 slot，完美解决这个问题~。

## uniapp 中部分标签的使用（life-follow）
在 uniapp 中`life-follow`无法使用。
```js
<life-follow
  a:if="{{show}}"
  sceneId="****"
  checkFollow="{{checkFollow}}"
  onCheckFollow="checkFollowCb"
  onClose="closeCb"
/>
```
被编译成
```
<life-follow
      vue-id="588c7fd8-1"
      sceneId=""
      checkFollow="{{checkLifeFlow}}"
      data-event-opts="{{[['^checkFollow',[['checkFollowCb']]],['close',[['closeCb']]]]}}"
      onCheckFollow="__e"
      onClose="__e"
      onVueInit="__l"
    ></life-follow>
```
添加 $event 参数后
```js
<life-follow
  a:if="{{show}}"
  sceneId="****"
  checkFollow="{{checkFollow}}"
  @checkFollow="checkFollowCb($event)"
  @close="closeCb($event)"
/>
```
编译为：
```js
<life-follow
      vue-id="588c7fd8-1"
      sceneId="***"
      checkFollow="{{checkLifeFlow}}"
      data-event-opts="{{[['^checkFollow',[['checkFollowCb',['$event']]]],['close',[['closeCb',['$event']]]]]}}"
      onCheckFollow="__e"
      onClose="__e"
      onVueInit="__l"
></life-follow>
```
为了区分自定义事件，uniapp, 添加 ^ 前缀，目前 uniapp 没有对 tag 进行区分，这部分最终走向了自定义事件，实际小程序中这里 js 报错。

### 解决办法
方法一：
使用小程序原生组件（非 uni 组件，mycomponents 这个目录下的小程序原生自定义组件），代码直接使用小程序原生的，不走 uniapp 的转换，规避这个问题。

方法二：
uniapp 给小程序特定标签加黑名单，这里不做自定义处理。我提了mr，官方觉得位置不好，换了个地方去写了：），好消息是你现在再用uniapp去新建一个项目，`life-follow`应该已经可以正常使用啦~

## uniapp debugger模式
在项目根目录新建 `.env` 文件,输入下面内容，即可开启 uniapp 的 DEBUG 模式，从控制台能看到不少有意思的东西。
```js
// .env
VUE_APP_DEBUG=true
```

## uniapp 的编译
@dcloudio/uni-template-compiler：uniapp 模板编译器，事件等处理都是在这里编译
@dcloudio/uni-mp-alipay：uniapp 平台运行时，平台相关的处理，事件函数的处理，一般都在这个文件中。

## uniapp 事件系统
```html
<view @click="query" />
```
会被编译成：
```html
<view data-event-opts="{{[['tap',[['query',['$event']]]]]}}"    onTap="__e" />
```
多个事件元素，它的`onTap`都是 `__e`，我们猜测，`__e`是 `uniapp` 事件系统的管理分发的角色，通过 `query`，找到调用者，参数是`$event`

### 编译事件代码分析
在解析模板之后，拿到相关事件，对once、capture等事件，添加特定前缀。事件处理，统一添加 `__e` 方法。
```js
function _processEvent(path, state, isComponent, isNativeOn = false, tagName, ret) {
    const opts = []
    // remove invalid event
    path.node.value.properties = path.node.value.properties.filter(property => {
        return property.key.value || property.key.name
    })
    const len = path.node.value.properties.length
    for (let i = 0; i < len; i++) {
        //  .... 省略
        const getEventType = state.options.platform.getEventType

        let optType = isCustom ? customize(type) : getEventType(type) // 比如自定义组件使用了 click 自定义事件

        //  添加前缀
        // VUE_EVENT_MODIFIERS: {
        //         capture: '!',
        //         once: '~',
        //         passive: '&',
        //         custom: '^'
        // },
        if (isOnce) {
            optType = VUE_EVENT_MODIFIERS.once + optType
        }
        if (isCustom) {
            optType = VUE_EVENT_MODIFIERS.custom + optType
        }
        opts.push({
            opt: t.arrayExpression([
                t.stringLiteral(optType),
                t.arrayExpression(methods)
            ]),
            params
        })

        keyPath.replaceWith(
            t.stringLiteral(
                state.options.platform.formatEventType(
                    isCustom ? customize(type) : getEventType(type), // 比如自定义组件使用了 click 自定义事件
                    isCatch,
                    isCapture,
                    isCustom
                )
            )
        )
        // INTERNAL_EVENT_PROXY === '__e' ,这里添加了 事件处理函数，'__e'
        valuePath.replaceWith(t.stringLiteral(INTERNAL_EVENT_PROXY))
    }
    return opts
}

```

### 事件调用代码分析
在页面初始化的时候，调用 parsePage，进行了事件绑定，`__e`指向了`handleEvent`
```js
function parsePage (vuePageOptions) {
  const [VueComponent, vueOptions] = initVueComponent(Vue, vuePageOptions)

  const pageOptions = {
    mixins: initBehaviors(vueOptions),
    data: initData(vueOptions, Vue.prototype),
    onLoad (query) {
      const properties = this.props

      const options = {
        mpType: 'page',
        mpInstance: this,
        propsData: properties
      }

      // 初始化 vue 实例
      this.$vm = new VueComponent(options)

      initSpecialMethods(this)

      // 触发首次 setData
      this.$vm.$mount()

      const copyQuery = Object.assign({}, query)
      delete copyQuery.__id__

      this.$page = {
        fullPath: '/' + this.route + stringifyQuery(copyQuery)
      }

      this.options = query
      this.$vm.$mp.query = query // 兼容 mpvue
      this.$vm.__call_hook('onLoad', query)
    },
    onReady () {
      initChildVues(this)
      this.$vm._isMounted = true
      this.$vm.__call_hook('mounted')
      this.$vm.__call_hook('onReady')
    },
    onUnload () {
      this.$vm.__call_hook('onUnload')
      this.$vm.$destroy()
    },
    events: {
      // 支付宝小程序有些页面事件只能放在events下
      onBack () {
        this.$vm.__call_hook('onBackPress')
      }
    },
    __r: handleRef,
    // 绑定事件
    __e: handleEvent,
    __l: handleLink$1,
    triggerEvent
  }

  initHooks(pageOptions, hooks$1, vuePageOptions)

  if (Array.isArray(vueOptions.wxsCallMethods)) {
    vueOptions.wxsCallMethods.forEach(callMethod => {
      pageOptions[callMethod] = function (args) {
        return this.$vm[callMethod](args)
      }
    })
  }

  return pageOptions
}
```
handleEvent 方法：
```js
function handleEvent (event) {
  event = wrapper$1(event)

  // [['tap',[['handle',[1,2,a]],['handle1',[1,2,a]]]]]
  const dataset = (event.currentTarget || event.target).dataset
  if (!dataset) {
    return console.warn('事件信息不存在')
  }
  const eventOpts = dataset.eventOpts || dataset['event-opts'] // 支付宝 web-view 组件 dataset 非驼峰
  if (!eventOpts) {
    return console.warn('事件信息不存在')
  }

  // [['handle',[1,2,a]],['handle1',[1,2,a]]]
  const eventType = event.type

  const ret = []

  eventOpts.forEach(eventOpt => {
    let type = eventOpt[0]
    const eventsArray = eventOpt[1]

    const isCustom = type.charAt(0) === CUSTOM
    type = isCustom ? type.slice(1) : type
    const isOnce = type.charAt(0) === ONCE
    type = isOnce ? type.slice(1) : type

    if (eventsArray && isMatchEventType(eventType, type)) {
      eventsArray.forEach(eventArray => {
        const methodName = eventArray[0]
        if (methodName) {
          let handlerCtx = this.$vm
          if (handlerCtx.$options.generic) { // mp-weixin,mp-toutiao 抽象节点模拟 scoped slots
            handlerCtx = getContextVm(handlerCtx) || handlerCtx
          }
          if (methodName === '$emit') {
            handlerCtx.$emit.apply(handlerCtx,
              processEventArgs(
                this.$vm,
                event,
                eventArray[1],
                eventArray[2],
                isCustom,
                methodName
              ))
            return
          }
          const handler = handlerCtx[methodName]
          if (!isFn(handler)) {
            throw new Error(` _vm.${methodName} is not a function`)
          }
          if (isOnce) {
            if (handler.once) {
              return
            }
            handler.once = true
          }
          let params = processEventArgs(
            this.$vm,
            event,
            eventArray[1],
            eventArray[2],
            isCustom,
            methodName
          )
          params = Array.isArray(params) ? params : []
          // 参数尾部增加原始事件对象用于复杂表达式内获取额外数据
          if (/=\s*\S+\.eventParams\s*\|\|\s*\S+\[['"]event-params['"]\]/.test(handler.toString())) {
            // eslint-disable-next-line no-sparse-arrays
            params = params.concat([, , , , , , , , , , event])
          }
          ret.push(handler.apply(handlerCtx, params))
        }
      })
    }
  })

  if (
    eventType === 'input' &&
    ret.length === 1 &&
    typeof ret[0] !== 'undefined'
  ) {
    return ret[0]
  }
}
```
代码有点长，但还是比较清晰的，函数对事件分类进行处理，最终事件函数被调用执行，也就是`handler.apply(handlerCtx, params)`,关键词 apply，寻找这个词，基本就能够找到函数的调用。




## 参考
  - [小程序插件 - uni-app官网](https://uniapp.dcloud.io/component/mp-weixin-plugin)
  - [fix: 支付宝小程序 life-follow 事件去掉修饰符 by Yaob1990 · Pull Request #2759 · dcloudio/uni-app](https://github.com/dcloudio/uni-app/pull/2759)
  - [fix:(mp-alipay): 支付宝小程序平台增加独有内置组件判断 #2410#issuecomment-878974559 · dcloudio/uni-app@92d682a](https://github.com/dcloudio/uni-app/commit/92d682a11a27d8fda573555512fff69a94a94e40)
  - [uni-app是如何构建小程序的？](https://juejin.cn/post/6968438754180595742#heading-22)