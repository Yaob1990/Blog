---
title: 封装小程序分页组件 waterflow-list
date: 2021-04-18 21:00:00
categories: 前端
img: ../../coverImages/mini-alipay.jpeg
tags:
  - 小程序
---

小程序开发中，列表分页组件是比较常见的需求，每次都要写 totalNum、pageSize、loading 状态等，写得多了，始终觉得很麻烦，萌生了封装一个容器组件的念头。

## 需求
1. 封装列表分页组件，包含加载状态
2. 尽量少的暴露接口，减少可配置项
3. 接口尽量少的同时，提供必要的数据重置方式，方便用户使用时候，稀奇古怪的需求

## 基础知识
### ref
[ref 获取组件实例 - 支付宝开放平台](https://opendocs.alipay.com/mini/framework/component-ref)
和vue中概念基本一致，可以通过 ref 获取实例的属性，从而修改组件数据的状态。

### slot
[组件模板和样式 - 支付宝开放平台](https://opendocs.alipay.com/mini/framework/component-template)
一般 slot 我们只是使用插入数据，很少通过 slot 获取内部数据。在支付宝小程序中可以通过作用域插槽 slot-scope ，实现外部对内部的数据访问。

## 组件实现
```html
<view>
  <slot name="item" list="{{list}}"></slot>
  <slot name="empty" a:if="{{init && !totalNum}}">
    <view>暂无数据</view>
  </slot>
  <slot name="loading" a:if="{{init && !finished}}">
    <view class="loading">
      <view class="text">加载中...</view>
    </view>
  </slot>
  <slot name="finished" a:if="{{ totalNum && finished}}">
    <view class="finished">
      <view class="line"></view>
      <view class="text">没有更多了</view>
      <view class="line"></view>
    </view>
  </slot>
</view>
```

```ts
import { Component } from 'herbjs';

interface IComponentData {}

interface IComponentProps {
  onGetData(currentPage: number, pageSize: number): void;

  pageSize: number;
}

interface IComponentMethods {}

interface IData {
  list: Array<any>;
  totalNum: number;
}

/**
 * Generics
 * @example
 *    Component<Data, Props, Methods, PageStore, AppStore>
 */
Component<IComponentData, IComponentProps, IComponentMethods>({
  data: {
    list: [],
    currentPage: 0, // 现在页面
    totalNum: 0, // 总条数
    totalPage: 0, // 总页数
    init: false,
    finished: false,
    loadingFlag: false,
  },
  props: {
    onGetData: () => {},
    pageSize: 20,
  },
  didMount() {},
  didUnmount() {},
  methods: {
    getData() {
      let { currentPage, finished, loadingFlag } = this.data;
      if (finished || loadingFlag) return;
      this.setData({
        loadingFlag: true,
      });
      return new Promise((resolve, reject) => {
        resolve(this.props.onGetData(++currentPage, this.props.pageSize));
      }).then((res: IData) => {
        console.log(res);
        const totalPage = Math.ceil(res.totalNum / this.props.pageSize);
        if (currentPage === 1) {
          this.setData({
            currentPage,
            list: res.list,
            init: true,
            totalPage,
            totalNum: res.totalNum,
            loadingFlag: false,
            finished:
              totalPage === 0 || (totalPage && currentPage === totalPage),
          });
        } else {
          this.setData({
            loadingFlag: false,
            currentPage,
            finished: totalPage && currentPage === totalPage,
            list: [...this.data.list, ...res.list],
          });
        }
      });
    },
  },
});

```



对外部暴露的接口

| 接口 | 作用 |
| --- | --- |
| slot name:item list | 需要循环的 list 数据 |
| slot name:loading  | 修改loading样式 |
| slot name:finished | 修改 finished 样式 |
| props onGetData| 获取外部数据  |
| props pageSize | 获取分页大小，默认 20 |
| Methods getData | 调用获取外部数据 |


### 使用：
```html
<view>
  <waterflow-list ref='saveComponentLoadData' onGetData="getListData">
    <view slot="item" slot-scope="props">
      <block a:for="{{props.list}}">
        <view>{{item.num}}</view>
      </block>
    </view>
  </waterflow-list>
</view>
```

```ts
Page<IPageState, IPageMethods, IPageStore>({
  onLoad(options) {
    this.init();
  },
  onReady() {
    this.addData();
  },
  onShow() {},
  onHide() {},
  onUnload() {},
  init() {
    this.dispatch('beforeRender');
  },
  // 保存 ref
  saveComponentLoadData(ref) {
    this.loadDataComponent = ref;
  },
  // 获取数据
  getListData(currentPage, pageSize) {
    return {
      list: [{ num: 1 }, { num: 2 }, { num: 3 }],
      totalNum: 3,
    };
  },
  // 手动触发获取数据
  addData() {
    this.loadDataComponent.getData();
  },
});
```

### 核心思想
通过获取组件 `ref`， 调用组件内部的 `getData` 方法，反射到主页面的`getListData`实现数据获取。
在通过 slot 的 slot-scope 获取需要展示的 list 数据，外部进行遍历并设置样式。
何时加载数据，由外部实现和决定，组件本身不会主动去请求数据。

### 注
1. 本文小程序指的是支付宝小程序，微信小程序好像并没有 slot-scope 功能
2. 框架使用 herbjs 

完整代码：
[mini-alipay-example/src/components/waterflow-list at main · Yaob1990/mini-alipay-example](https://github.com/Yaob1990/mini-alipay-example/tree/main/src/components/waterflow-list)
