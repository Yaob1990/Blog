---
title: 11 月的碎碎念
categories:
  - anything
date: 2020-11-29 21:33:36
tags:
---



1. 启用 jscoder.com 域名

   一直对现在的域名（aocoding.com）不是非常满意，但是又没有合适的域名。最近把 jscoder.com 续费10年，长期使用。希望自己多多努力，产出文章，不负自己的 money 投入。

2. 购入 4c8g5m 机器

   前期的两台1c2g1m 小机器，给我带来了不少的快乐，然而，性能越来越满足不了我的需要。我想试试docker，想试试资源本地部署，不走cdn，这些小机器都无法满足。

   现在小机器上面跑了1个爬虫，api 服务，每次ssh部署 node 服务就难受，要等待好久。

   现在大机器上，部署docker，node 都很快，不要把生命浪费在等待上，该换就换，该升级就升级。机器价值是有限的，而你提供的服务是无价的：）

3. 学习 docker

   docker 大法好，docker 大法妙，docker 呱呱叫。

   然鹅，部署项目时候，还是选择了宝塔面板，实在是太易用了...不要考虑那么多，无脑操作，项目就部署好了。待我 docker 再好好学一学，再考虑直接换成 docker 咯....

   k8s 实在是太复杂了，项目搭建都一堆问题搞不定，算啦算啦，现在 docker-compose 已经能满足我的需求了。

4. miAxios 经历项目考验

   封装的小程序请求库  [miniAxios](https://github.com/Yaob1990/miniAxios#readme)  在支付宝小程序项目中运行良好，没有严重bug。

   request 库的封装，参考了 axios，axios ts 教程，等等内容，封装虽然没有什么技术含量，自己还是学到了很多东西，加深了对 ts 的理解。

   后续计划：

   1. 加强对多平台的支持，目前虽然是支持多平台的，但是我自己也没有完整测试过=-=。
   2. 加强 ts 类型判断，目前的 ts 在 Response 部分还是有点问题。
   3. 统一返回参数。目前的 response 是根据返回报文直接返回，后续，会归集到 response 的 data 下面，类似： res.config.data 的结构

