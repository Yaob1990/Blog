---
title: herbjs 简明数据流
date: 2020-08-16 16:29:42
categories: 前端
img: ../../coverImages/zhifubao-mini.png
tags:
  - 支付宝小程序,herbjs
typora-root-url: ..\..
---

最近在使用`herbjs`做支付宝小程序的开发，在熟悉了开发模式之后，还是挺好用的。



### 小程序运行机制

写小程序的感觉，更像是在写ract，然而这两者的机制是不一样的。

小程序的框架包含两部分View视图层(可能存在多个)、App Service逻辑层(一个)，View层用来渲染页面结构，AppService层用来逻辑处理、数据请求、接口调用，它们在两个线程里运行。

视图层使用WebView渲染，逻辑层使用JSCore运行。

视图层和逻辑层通过系统层的JsBridage进行通信，逻辑层把数据变化通知到视图层，触发视图层页面更新，视图层把触发的事件通知到逻辑层进行业务处理。

<img src="/images/image-20200816164049483.png" alt="image-20200816164049483" style="zoom:50%;" />

小程序的`setData`并不是实时的，而是需要两个线程通过 `JsBridge` 进行通信完成，在性能优化方面，需要考虑：

1. 避免频繁的 `setData`，容易造成卡顿，渲染出现延迟
2. 避免`setData`频繁传递大量数据
3. 避免后台页面`setData`,用户无法感知，会抢占前台页面的执行



### herbjs 与原生开发框架相比

原生框架数据以page为单位，进行管理。

herbjs 吸收了 vuex 的精华，给小程序增加了，全局 store，页面store，等一系列的 vuex 功能。还支持 `typescript`,`插件体系`等功能。



区别于 `uni-app`,`taro`,这些框架，herbjs 并没有修改小程序本身的逻辑，原有的生命周期都还在，原有的写法也都适用，更多的可以看成是原生小程序的增强版。

个人觉得，`简明的数据流`是这个框架最优雅的地方，方便大量 vue 开发者快速上手开发。

官方文档: https://www.yuque.com/herbjs/doc

官方文档，其实说的已经比较明白了，值得多看几遍。



###  简明数据流

项目地址：https://github.com/Yaob1990/leran_herbjs

#### 页面级别数据流

页面中，通过 `setData`数据

```typescript
Page<IPageState, IPageMethods, IPageStore>({
  onLoad(options) {
    this.init();
  },
  data: {
    num: 0,
  },
  plus() {
    this.setData({
      num: ++this.data.num,
    });
  },
  minus() {
    this.setData({
      num: --this.data.num,
    });
  },
});
```

#### 组件中数据流

外部传入参数，注意如果是监听事件必须是`on`开头

```typescript
Component<IComponentData, IComponentProps, IComponentMethods>({
  mapStateToData: [],
  data: {
    text: 'component',
  },
  props: {
    onPlus: () => {},
    onMinus: () => {},
  },
  didMount() {},
  didUnmount() {},
  methods: {
    plus() {
      this.props.onPlus();
    },
    minus() {
      this.props.onMinus();
    },
  },
});
```

组件使用：

```html
<action onPlus="plus" onMinus="minus"></action>
```



#### 页面 store 数据流

```typescript
Page<IPageData, IPageMethods, IPageStore>({
  plus() {
    this.commit('plus');
  },
  minus() {
    this.commit('minus');
  },
  multiply() {
    this.dispatch('multiplyAsync');
  },
});
```



store 部分

```typescript

Store<IPageState, IPageGetters, IPageMutations, IPageActions>({
  state: {
    num: 0,
  },
  // 全局 Getter 可以被 App、Page、Component 都能访问到
  // 页面 Getter 只能被当前 Page 和 当前 Page 内的 Component 访问到
  getters: {
    desc: ({ state, getters }) => {
      if (state.num > 0) {
        return '正值';
      } else if (state.num < 0) {
        return '负值';
      } else {
        return '零';
      }
    },
  },
  mutations: {
    plus(state) {
      state.num = ++state.num;
    },
    minus(state) {
      state.num = --state.num;
    },
    multiply(state) {
      state.num = state.num * 10;
    },
  },
  actions: {
    async multiplyAsync({ state, commit, dispatch }, payload) {
      setTimeout(() => {
        commit('multiply');
      }, 5000);
    },
  },
});
```



#### 全局 store 

全局 store 和页面类似，只是多一个 `$global`参数，详见文档。



### 开发体会

小程序的开发框架百花齐放，uni-app，taro，mpvue。

然而，自己也只是停留在会用的程度，没有再去想一想为什么可以这样编译，内部的原理。这阶段的工作估计会长期和小程序打交道，希望自己能够深入的研究小程序的背后逻辑，而不仅仅会用就行。

道阻且长，同志加油。









