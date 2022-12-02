---
title: weex兼容本地图片
categories:
  - front-end
tags:
  - weex
  - 混合开发
date: 2019-07-05 05:55:00
---

做 weex 开发时候，不能使用本地图片，比较难受，必须使用网络图片。
那么能不能本地开发的图片自动转成网络图片？

### 兼容本地图片实现

本地图片都存放在/src/image 文件夹，通过 tool.getImage("logo.png") 访问图片。在运行和部署时候，图片地址转成网络地址。
适合 web，不适合 app 场景。

### demo

[weex-local-image](https://github.com/aocoding/weex-local-image)
