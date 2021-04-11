---
title: webstorm 中调试typescript单文件
date: 2020-05-22 10:56:35
categories: 前端
img: ../../coverImages/ts-node.svg
tags:
  - typescript
typora-root-url: ..\..
---





leetCode 最近支持了`typescript`刷题，高高兴兴把`webstorm`切换到typescript刷题,记录下切换过程。

## 安装

因为刷题都是单文件，我们需要用webstorm支持debugger单ts文件。

`yarn add typescript ts-node`  ts-node 可以让我们直接执行ts文件



## 配置

1. 配置部分 Node parameters 填写：`--require ts-node/register`。
2. 点击运行，就可以直接执行`.ts`单文件，点击debugger就可以进行`debugger`了，和以前调试js一样，非常方便



![配置](/images/image-20200522110323174.png)

![运行](/images/image-20200522110831517.png)