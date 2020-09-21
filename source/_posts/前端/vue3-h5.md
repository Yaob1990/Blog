---
title: vue3 h5 脚手架
date: 2020-09-21 22:22:22
categories: 前端
img: ../../coverImages/vue-3.jpg
tags:
  - vue
typora-root-url: ..\..

---



vue 3 终于发布了正式版。

也把自己用了很久的 h5 脚手架更新了一波，所有依赖都升级到最新，后续h5开发就直接使用最新的脚手架进行开发。

后续还会增加多页面的脚手架，多页面写的真的不多，但是配置还是有点意思的~

本次脚手架，抛弃了单元测试，业务太多，真的写不过来=-=

Node-sass 换成了less，不多说，懂得自然懂....



地址：https://github.com/Yaob1990/vue3-h5



## 主要功能

      1. 集成最新 `vue` 全家桶，使用 `typescript` 开发
      2. 路由使用 hash 模式，方便部署
      3. `axios` 封装，暴露常用的 `get` `post` 方法
      4. `axios` 封装，避免多重 `loading` 的问题，并提供接口配置项
      5. 使用 less ，提供基础的`mixin`，并作为全局css，方便使用
      6. 异步加载 `vconsole`，线上环境，无需手动去除，不会打入主包，不会引起包体积增大
      7. ui 框架选择 `vant`，按需引入
      8. `npm run report` 分析构建包的大小
      9. 线上版本去除 console.log，debugger 等调试内容







