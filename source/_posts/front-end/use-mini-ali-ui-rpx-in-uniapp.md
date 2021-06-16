---
title: 在 uniapp 中使用小程序ui组件库（mini-ali-ui-rpx）
date: 2021-06-17 06:00:00
categories: 前端
tags:
  - 小程序
img: ../../coverImages/mini-alipay.jpeg
---

uniapp 提供了很多开箱即用的特性，但是业务开发中，弹框之类好像很少使用原生的小程序组件，这个特性其实是支持的。从 uniapp 开发者角度，支持原生小程序特性有利于开发者对现有的小程序做快速迁移，能够兼容社区绝大多数 ui 组件库。

## 兼容操作（以支付宝小程序为例）
1. 在 src 目录中新建 `wxcomponents/mycomponents` 文件夹，用于存放小程序组件
2. 使用：在 pages.json 的 globalStyle -> usingComponents 引入组件，注意这里必须是全局而不是页面级别。
    ![](/images/16238837739805.jpg)

3. 在页面中使用
    ```
    <!-- 页面模板 (index.vue) -->
    <view>
        <!-- 在页面中对自定义组件进行引用 -->
        <custom name="uni-app"></custom>
    </view>
    ```
    
    ## 引入 mini-ali-ui-rpx
    1. 通过 npm 下载 `mini-ali-ui-rpx` 文件
    2. 复制 `mini-ali-ui-rpx` 到 `mycomponents` 文件夹下
    3. 在 pages.json 的 globalStyle -> usingComponents 引入组件，注意如果使用的组件依赖一些基础组件，如`am-icon/am-button`，这些组件必须也要被引入，否则找不到组件，报错。
代码地址：[Yaob1990/use-mini-ali-ui-rpx-in-uniapp: 在 uniapp 中使用 mini-ali-ui-rpx](https://github.com/Yaob1990/use-mini-ali-ui-rpx-in-uniapp)

在线预览：https://sandbox.jscoder.com/use-mini-ali-ui-rpx-in-uniapp

## 参考：
* [框架简介 - uni-app官网](https://uniapp.dcloud.io/frame?id=%e5%b0%8f%e7%a8%8b%e5%ba%8f%e8%87%aa%e5%ae%9a%e4%b9%89%e7%bb%84%e4%bb%b6%e6%94%af%e6%8c%81)