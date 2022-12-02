---
title: 深入理解支付宝小程序 -- 准备篇
categories:
  - front-end
img: ../../coverImages/mini-alipay.jpeg
tags:
  - 小程序
date: 2021-07-31 21:00:00
---

> 深入理解总是困难的，耐心和寂寞相伴，沉默和思考相随，进一退三，推着我们前进，是对未知的渴望，是残存的技术热情。

本系列，将开始对支付宝小程序的运行时的分析，网上资料比较少，一些猜想不一定对，我姑且说之，您姑且听之。因为微信小程序的先驱性，会对两者的部分实现进行对比。

## ide vs miniu
ide 和 miniu 各有特色，各自维护了一套小程序的运行时，调试过程中两者其实都有使用到。

miniu 可以直观的看到它的构建产物，一般在 `.miniu/plugins/dev/dist/main` 目录下构建产物我们可以直接修改，再次运行。

ide 是一个正统项目，他是根据 `vscode` 修改过来的，因为 `vscode` 使用 `electron`  实现，我们可以使用 `electron` 去窥探小程序的实现。

## 解包 vol_modules.asar
vol_modules.asar 路径：`/Applications/小程序开发者工具.app/Contents/Resources/app/vol_modules.asar`
`asar`是 `electron` 的编译文件，我们可以尝试对它进行反编译：
```shell
// 安装
npm install -g asar

// 解包
asar extract vol_modules.asar ./vol_modules/
```
它其实是一个 node_modules 的包，内部有众多的包，后续我们主要分析 `lyra-integration-ide` 包，它的名字叫做：小程序模拟器集成。

## ide 开发者模式
ide 是基于 `electron`,我们可以打开它的开发者模式：
![](/images/16277299421308.jpg)

在打开的控制台中输入：`document.getElementsByTagName('webview')[1].openDevTools()`

我们这里就打开了小程序渲染之后页面级别的页面。

## mini-pkg-builder
`mini-pkg-builder`，这个程序在 miniu 和 ide 都有使用到，路径稍有不同：
miniu：`/Users/tom/.miniu/compiler/tiny6.3.0_cube0.49.18/mini-pkg-builder`,
ide: `/Applications/小程序开发者工具.app/Contents/Resources/app/builder/mini-pkg-builder`

这个程序本质上是小程序的打包程序，尝试对其输入进行重定向输出：

```shell
#!/bin/bash
echo "$*" >> /Users/tom/.miniu/compiler/tiny6.3.0_cube0.49.18/mini-pkg-builder.txt
```

拦截之后得到：

```shell
// ide
--input /Users/tom/MiniProjects/blank --output /var/folders/p1/9gklj64n229bys726brqv6040000gn/T/.miniu_dist/ng-main --zrender --prod --no-vuerender --output-code-placeholder --sourcemap false --cheap build --thread-pool-timeout 500 --cache-directory /Users/tom/MiniProjects/blank/.miniu/cache --target web --appxcompatible tiny --external appx --mode concurrent --modulize-minify process --modulize-minify-concurrent 7 --project-config-json %7B%22enableAppxNg%22%3Atrue%7D --stat-flag tag,meta --stat-output-path /var/folders/p1/9gklj64n229bys726brqv6040000gn/T/.miniu_dist/ng-main.json --webpackstats /var/folders/p1/9gklj64n229bys726brqv6040000gn/T/.miniu_dist/ng-webpack-stat.json
--input /Users/tom/MiniProjects/blank --output /Users/tom/MiniProjects/blank/.miniu/plugins/dev/dist/ng-main --zrender --no-vuerender --sourcemap --cheap build --force-sourcemap --thread-pool-timeout 500 --cache-directory /Users/tom/MiniProjects/blank/.miniu/cache --target web --appxcompatible tiny --external appx --mode concurrent --modulize-minify process --modulize-minify-concurrent 7 --project-config-json %7B%22enableAppxNg%22%3Atrue%7D --watch --no-minify --output-code-placeholder --notify-flag lifecycle --notify-type ipc
// miniu
--input /Users/tom/MiniProjects/blank --output /Users/tom/MiniProjects/blank/.miniu/plugins/dev/dist/ng-main --zrender --no-vuerender --sourcemap --cheap build --force-sourcemap --thread-pool-timeout 500 --cache-directory /Users/tom/MiniProjects/blank/.miniu/cache --target web --appxcompatible tiny --external appx --mode concurrent --modulize-minify process --modulize-minify-concurrent 7 --project-config-json %7B%22enableAppxNg%22%3Atrue%7D --watch --no-minify --output-code-placeholder --notify-flag lifecycle --notify-type ipc
```
这里其实就是小程序的编译命令，它本质是是多入口的 webpack 编译，常用参数 `sourcemap` ,`target web` 挺眼熟的。

## 构建产物
这里是单入口的页面，多入口类似，为了方便分析，我们一般会控制变量，只保留最简单的功能。
.
├── appConfig.json
├── assets
│   └── logo.png
├── index.html
├── index.js
├── index.worker.html
├── index.worker.js
└── manifest.json

## 参考

- [小程序底层实现原理及一些思考 - 知乎](https://zhuanlan.zhihu.com/p/81775922)
- [小程序底层实现原理及一些思考（2） - 知乎](https://zhuanlan.zhihu.com/p/121815358)
- [独家！支付宝首次披露其小程序技术架构](https://mp.weixin.qq.com/s/PX7b_qV6tYKnN3ecoz9Ehw)
- [利用sourceMap定位错误实践](https://juejin.cn/post/6882265367251517447)
- [微信小程序开发专区 | 薛定喵君](http://tiaocaoer.com/xcx_study/)
